---
theme: dashboard
title: Current Results
toc: false
---


# Recon Analysis Over different Model Types

```js
// ---------- Imports ----------
const [THREE, {STLLoader}, {OrbitControls}] = await Promise.all([
  import("https://esm.sh/three@0.160.0"),
  import("https://esm.sh/three@0.160.0/examples/jsm/loaders/STLLoader.js?deps=three@0.160.0"),
  import("https://esm.sh/three@0.160.0/examples/jsm/controls/OrbitControls.js?deps=three@0.160.0")
]);
const {MeshBVH, acceleratedRaycast} = await import(
  "https://esm.sh/three-mesh-bvh@0.7.3?deps=three@0.160.0"
);

// ---------- Brain mesh (Promise<Mesh>) ----------
const brainMesh = (async () => {
  const loader = new STLLoader();
  const buffer = await FileAttachment("data/MNI152_1mm_brain.stl").arrayBuffer();
  const geometry = loader.parse(buffer);
  geometry.boundsTree = new MeshBVH(geometry);

  const material = new THREE.MeshPhongMaterial({
    color: 0xaaaaaa,
    transparent: true,
    opacity: 0.15,
    depthWrite: false,
    side: THREE.DoubleSide
  });

  const mesh = new THREE.Mesh(geometry, material);
  mesh.raycast = acceleratedRaycast;
  return mesh;
})();
```

```js
// ---------- Helpers ----------

const uiState = {onlyInside: false};

function getElectrodes(data, {valueKey = "corr"} = {}) {
  return data.nodes
    .filter(n => !(n.x === 0 && n.y === 0 && n.z === 0))
    .map(n => ({...n, value: n[valueKey]}));
}

function flagInside(electrodes, mesh) {
  const rc = new THREE.Raycaster();
  const dir = new THREE.Vector3(0.577, 0.577, 0.577).normalize();
  mesh.updateMatrixWorld();
  return electrodes.map(e => {
    rc.set(new THREE.Vector3(e.x, e.y, e.z), dir);
    return rc.intersectObject(mesh, false).length % 2 === 1;
  });
}

// Persistent 3D view — one canvas shared between renderer and controls.
function makeBrainView(mesh, {width = 520, height = 480, domain = [-0.8, 0.8]} = {}) {
  const scene = new THREE.Scene();
  scene.background = new THREE.Color(0xffffff);
  scene.add(mesh.clone());
  scene.add(new THREE.AmbientLight(0xffffff, 1));
  const dl = new THREE.DirectionalLight(0xffffff, 0.8);
  dl.position.set(1, 1, 1);
  scene.add(dl);

  const color = d3.scaleSequential(d3.interpolateViridis).domain(domain);
  const sphereGeo = new THREE.SphereGeometry(2, 8, 8);
  const sphereMat = new THREE.MeshBasicMaterial();
  const hidden = new THREE.Matrix4().makeScale(0, 0, 0);
  let inst = null, positions = [];

  const setElectrodes = electrodes => {
    if (inst) { scene.remove(inst); inst.dispose(); }
    inst = new THREE.InstancedMesh(sphereGeo, sphereMat, electrodes.length);
    positions = electrodes.map(e => new THREE.Matrix4().setPosition(e.x, e.y, e.z));
    electrodes.forEach((e, i) => {
      inst.setMatrixAt(i, positions[i]);
      inst.setColorAt(i, new THREE.Color(color(e.value)));
    });
    scene.add(inst);
  };
  const setVisible = flags => {
    if (!inst) return;
    positions.forEach((p, i) => inst.setMatrixAt(i, !flags || flags[i] ? p : hidden));
    inst.instanceMatrix.needsUpdate = true;
  };

  const camera = new THREE.PerspectiveCamera(45, width / height, 1, 2000);
  camera.up.set(0, 0, 1);
  camera.position.set(350, -17, 6);

  const canvas = document.createElement("canvas");
  canvas.width = width;
  canvas.height = height;

  const renderer = new THREE.WebGLRenderer({canvas, antialias: true});
  renderer.setSize(width, height);
  renderer.setPixelRatio(window.devicePixelRatio || 1);

  const orbit = new OrbitControls(camera, canvas);
  orbit.target.set(0, -18, 6);
  orbit.enableDamping = true;

  renderer.setAnimationLoop(() => {
    orbit.update();
    renderer.render(scene, camera);
  });

  window.addEventListener("pagehide", () => {
    renderer.setAnimationLoop(null);
    orbit.dispose();
    renderer.dispose();
  }, {once: true});

  return {element: canvas, setElectrodes, setVisible};
}

// Plots (returns nodes, so they can be re-rendered imperatively)
function makeDistPlots(data, {xDomain = [-0.4, 1], label = "Correlation", colorDomain = [-0.8, 0.8], yMax = 0.12} = {}) {
  const colorOpts = {type: "sequential", scheme: "Viridis", domain: colorDomain, label};

  const hist = Plot.plot({
    width: 420, height: 300, marginLeft: 50, marginRight: 20, marginBottom: 20, marginTop: 20,
    x: {domain: xDomain, label: null},
    y: {label: "Proportion", grid: true, domain: [0, yMax]},
    color: colorOpts,
    marks: [
      Plot.rectY(data, Plot.binX({y: "proportion", fill: "mean"}, {x: "value", fill: "value", thresholds: 40})),
      Plot.ruleX([0], {stroke: "black", strokeDasharray: "3,3"}),
      Plot.ruleY([0])
    ]
  });

  const box = Plot.plot({
    width: 420, height: 90, marginTop: 0, marginLeft: 50, marginRight: 20,
    x: {domain: xDomain, label, grid: true},
    y: {axis: null},
    marks: [
      Plot.boxX(data, {x: "value", stroke: "#444", fill: "#ddd"}),
      Plot.ruleX([0], {stroke: "black", strokeDasharray: "3,3"})
    ]
  });

  const colorbar = Plot.legend({color: colorOpts, width: 320, ticks: 5});

  const title = html`<div>
    <div style="font-size:16px;font-weight:bold">Distribution of Electrode Reconstruction ${label}s</div>
    <div style="font-size:12px;color:#666">Share of electrodes per bin (n = ${data.length})</div>
  </div>`;

  return {title, plots: html`<div>${hist}${box}</div>`, colorbar};
}

// Dashboard around a persistent `view`
function electrodeDashboard({data, mesh, view, valueKey = "corr", label = "Correlation", ...opts}) {
  const electrodes = getElectrodes(data, {valueKey});
  let inside = null;
  const getInside = () => inside ??= flagInside(electrodes, mesh);
  view.setElectrodes(electrodes);

  const checkbox = html`<input type="checkbox">`;
  checkbox.checked = uiState.onlyInside;
  const status = html`<span></span>`;
  const titleBox = html`<div style="text-align:center"></div>`;
  const colorbarBox = html`<div style="display:flex;justify-content:center;margin:4px 0 8px"></div>`;
  const plotsBox = html`<div style="flex:none"></div>`;

  const update = () => {
    const only = checkbox.checked;
    uiState.onlyInside = only;
    const flags = only ? getInside() : null;
    view.setVisible(flags);
    const shown = only ? electrodes.filter((_, i) => flags[i]) : electrodes;
    status.textContent = `Showing ${shown.length} of ${electrodes.length} electrodes`;
    const {title, plots, colorbar} = makeDistPlots(shown, {label, ...opts});
    titleBox.replaceChildren(title);
    colorbarBox.replaceChildren(colorbar);
    plotsBox.replaceChildren(plots);
  };
  checkbox.addEventListener("input", update);
  update();

  const node = html`<div style="font-family:sans-serif;width:max-content;max-width:100%">
    <label style="font-size:14px;display:block;margin-bottom:8px">${checkbox} Only show electrodes inside the mesh · ${status}</label>
    ${titleBox}
    ${colorbarBox}
    <div style="display:flex;gap:12px;align-items:flex-start;overflow-x:auto">
      <div style="flex:none">${view.element}</div>
      ${plotsBox}
    </div>
  </div>`;
  return Object.assign(node, {electrodes, getInside});
}
```

```js
// Build the 3D view once.
const brainView = makeBrainView(await brainMesh);
```

```js
// ---------- File registry + tokens ----------

const registry = {
  "recon_results_task-all_motifsubs_motifFalse.json": FileAttachment("data/json_recon_data/recon_results_task-all_motifsubs_motifFalse.json"),
  "recon_results_task-all_motifsubs_motifTrue.json": FileAttachment("data/json_recon_data/recon_results_task-all_motifsubs_motifTrue.json"),
  "recon_results_task-film_motifsubs_fmriOnly.json": FileAttachment("data/json_recon_data/recon_results_task-film_motifsubs_fmriOnly.json"),
  "recon_results_task-film_motifsubs_motifFalse.json": FileAttachment("data/json_recon_data/recon_results_task-film_motifsubs_motifFalse.json"),
  "recon_results_task-film_motifsubs_motifTrue.json": FileAttachment("data/json_recon_data/recon_results_task-film_motifsubs_motifTrue.json"),
  "recon_results_task-film_motifsubs_wac_motifFalse.json": FileAttachment("data/json_recon_data/recon_results_task-film_motifsubs_wac_motifFalse.json"),
  "recon_results_task-film_motifsubs_wac_motifTrue.json": FileAttachment("data/json_recon_data/recon_results_task-film_motifsubs_wac_motifTrue.json"),
  "recon_results_task-music_motifsubs_fmriOnly.json": FileAttachment("data/json_recon_data/recon_results_task-music_motifsubs_fmriOnly.json"),
  "recon_results_task-music_motifsubs_motifFalse.json": FileAttachment("data/json_recon_data/recon_results_task-music_motifsubs_motifFalse.json"),
  "recon_results_task-music_motifsubs_motifTrue.json": FileAttachment("data/json_recon_data/recon_results_task-music_motifsubs_motifTrue.json"),
  "recon_results_task-speech_motifsubs_fmriOnly.json": FileAttachment("data/json_recon_data/recon_results_task-speech_motifsubs_fmriOnly.json"),
  "recon_results_task-speech_motifsubs_motifFalse.json": FileAttachment("data/json_recon_data/recon_results_task-speech_motifsubs_motifFalse.json"),
  "recon_results_task-speech_motifsubs_motifTrue.json": FileAttachment("data/json_recon_data/recon_results_task-speech_motifsubs_motifTrue.json"),
  "recon_results_task-speech_motifsubs_wac_motifFalse.json": FileAttachment("data/json_recon_data/recon_results_task-speech_motifsubs_wac_motifFalse.json"),
  "recon_results_task-speech_motifsubs_wac_motifTrue.json": FileAttachment("data/json_recon_data/recon_results_task-speech_motifsubs_wac_motifTrue.json"),
  "recon_results_task-music_motifsubs_wac_motifFalse.json": FileAttachment("data/json_recon_data/recon_results_task-music_motifsubs_wac_motifFalse.json"),
  "recon_results_task-music_motifsubs_wac_motifTrue.json": FileAttachment("data/json_recon_data/recon_results_task-music_motifsubs_wac_motifTrue.json")
};

const tokens = {
  subjects: {"Motif subjects": "motifsubs"},
  task: {"All": "all", "Rest": "rest", "Film": "film", "Film (speech)": "speech", "Film (music)": "music"},
  motif: {"EEG + fMRI": "motifTrue", "EEG": "motifFalse", "fMRI": "fmriOnly"},
  wac: {"No": "", "Yes": "_wac"}
};

const modelKey = {"Within": "within", "Across": "across"};

const baseName = s =>
  `recon_results_task-${tokens.task[s.task]}_${tokens.subjects[s.subjects]}${tokens.wac[s.wac]}_${tokens.motif[s.motif]}`;
const findFile = s => registry[baseName(s) + ".json"];
```

```js
// ---------- Menu ----------
// NOTE: we keep the form's value in a small local state object and wire
// `input` listeners ourselves. This avoids Generators.input(sel), which
// triggers "element.addEventListener is not a function" with Inputs@0.12.

const state = {
  subjects: "Motif subjects",
  task: "Film (speech)",
  motif: "fMRI",
  wac: "No",
  model: "Within"
};

const listeners = new Set();
const onChange = fn => listeners.add(fn);
const fireChange = () => listeners.forEach(fn => fn(state));

function makeSelect(options, key, label) {
  const s = Inputs.select(options, {label, value: state[key]});
  s.addEventListener("input", () => { state[key] = s.value; fireChange(); });
  return s;
}

function makeRadio(options, key, label) {
  const r = Inputs.radio(options, {label, value: state[key]});
  r.addEventListener("input", () => { state[key] = r.value; fireChange(); });
  return r;
}

const sel = Inputs.form({
  subjects: makeSelect(Object.keys(tokens.subjects), "subjects", "Subjects"),
  task: makeSelect(Object.keys(tokens.task), "task", "Task"),
  motif: makeSelect(Object.keys(tokens.motif), "motif", "Motif"),
  wac: makeRadio(Object.keys(tokens.wac), "wac", "WAC"),
  model: makeRadio(Object.keys(modelKey), "model", "Model")
});

display(sel);
```

```js
// ---------- Reactive dashboard (imperative redraw) ----------
// `container` is stable; we call `redraw()` on every menu change and on
// page load, replacing the dashboard node each time.

const container = html`<div style="min-height:200px"></div>`;
display(container);

async function redraw() {
  const s = {...state};
  const wanted = baseName(s);
  const file = findFile(s);

  container.replaceChildren(html`<div style="font-family:sans-serif;color:#666;padding:8px">Loading…</div>`);

  if (!file) {
    container.replaceChildren(html`<div style="font-family:sans-serif;padding:12px;border:1px solid #e0a;background:#fff5fa">
      No file available for <code>${wanted}.json</code>
      (Task: ${s.task}, Motif: ${s.motif}, WAC: ${s.wac}).
    </div>`);
    return;
  }

  let raw;
  try {
    raw = await file.json();
  } catch (err) {
    container.replaceChildren(html`<div style="font-family:sans-serif;padding:12px;border:1px solid #e0a;background:#fff5fa">
      Failed to read <code>${wanted}.json</code>: ${err.message}
    </div>`);
    return;
  }

  const m = modelKey[s.model];
  const corr = raw[`${m}_corr`];
  const locs = raw[`${m}_locs`];
  const validLocs =
    Array.isArray(locs) &&
    locs.length > 0 &&
    locs.every(l => Array.isArray(l) && l.length === 3);

  if (!validLocs) {
    container.replaceChildren(html`<div style="font-family:sans-serif;padding:12px;border:1px solid #e0a;background:#fff5fa">
      The <code>${m}_locs</code> field in <code>${wanted}.json</code>
      doesn't contain [x, y, z] coordinates
      (first entry: <code>${JSON.stringify(locs?.[0])}</code>),
      so the ${s.model} model can't be plotted in 3D.
      The file needs to be regenerated.
    </div>`);
    return;
  }

  const selData = {
    nodes: locs.map(([x, y, z], i) => ({id: i, x, y, z, corr: corr[i]}))
  };

  const node = electrodeDashboard({
    data: selData,
    mesh: await brainMesh,
    view: brainView
  });
  container.replaceChildren(node);
}

onChange(redraw);
redraw();
```
<center>
  <span style="font-size: 0.6em;">
    <font color="grey">Figure # displayes reconstruction of electrodes and how they correlate to their emperical values. WAC is within across combination which is represented by the acorss model when the WAC option is selected. Motif EEG refers to both EEG, ECoG, and iEEG.</font>
  </span>
</center>
