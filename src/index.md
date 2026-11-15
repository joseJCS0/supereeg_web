---
toc: false
---

```js
const pdfUrl = FileAttachment("./data/CNS_supereeg_poster_v3.pdf").href;
const gifUrl = FileAttachment("./data/example_model.gif").href;
const start_upUrl = FileAttachment("./data/supereeg_pipeline_startup-1.pdf").href;
const embed = document.getElementById("pdf-embed");
embed.setAttribute("data", pdfUrl);
const embed2 = document.getElementById("pdf-embed1");
embed2.setAttribute("data", start_upUrl);
document.getElementById("pdf-download").setAttribute("href", pdfUrl);
document.getElementById("pdf-download1").setAttribute("href", start_upUrl);
document.getElementById("hero-gif").setAttribute("src", gifUrl);
```

<div class="hero">
  <h1>SuperEEG: A Gaussian Model of the Human Brain</h1>
  <img id="hero-gif" alt="SuperEEG animation" width="600" style="margin: 1rem 0; border-radius: 8px;">
  <p>SuperEEG is a gausian process model developed by Lucy L W Owen and her team from Darthmouth intended to model the human brain. This framework will focus on model optimizations/changes that Jose J Carmona-Sanchez introduced. Specfically this framwork will be showcasing inclusion of both electrophysological and MRI data.<br><br>Inution and introduction of the model will be talked about in this framwork but for a deeper introduction please refer to the initial write up by Lucy L W Owen and her team. There is also a poster down below introducing intial optmization steps that were taken.</p>
  
  <a href="https://academic.oup.com/cercor/article/30/10/5333/5851264?__cf_chl_tk=Vs_JYH4EcOc2U5sum0KYfPVIE7jqZenRjb2JQDGtQiU-1765925269-1.0.1.1-o3YexL2QwzmC4KL2PrZtLyAzuh03DbacW.EXaAcY90E">The initial writeup by Lucy L W Owen and her team can be found here<span style="display: inline-block; margin-left: 0.25rem;">↗︎</span></a>

  <object id="pdf-embed" type="application/pdf" width="100%"    height="600px" style="margin: 1rem 0;">
  <p>Your browser cannot display this PDF. <a id="pdf-download" href="#">Click here to download</a>.</p>
  </object>

  <p>Down below is a guide to using the current SuperEEG.</p>
  
  <object id="pdf-embed1" type="application/pdf" width="100%"    height="600px" style="margin: 1rem 0;">
  <p>Your browser cannot display this PDF. <a id="pdf-download1" href="#">Click here to download</a>.</p>
  </object>

</div>

<style>

.hero {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: var(--sans-serif);
  margin: 4rem 0 8rem;
  text-wrap: balance;
  text-align: center;
}

.hero h1 {
  margin: 1rem 0;
  padding: 1rem 0;
  max-width: none;
  font-size: 12vw;
  font-weight: 900;
  line-height: 1;
  background: linear-gradient(30deg, var(--theme-foreground-focus), currentColor);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.hero h2 {
  margin: 0;
  max-width: 34em;
  font-size: 20px;
  font-style: initial;
  font-weight: 500;
  line-height: 1.5;
  color: var(--theme-foreground-muted);
}

@media (min-width: 640px) {
  .hero h1 {
    font-size: 75px;
  }
}

</style>
