---
title: Introduction & Intuition
---

```js
const imgUrl = FileAttachment("./data/tech_resolution.jpg").href;
const pngUrl = FileAttachment("./data/Dropped_Image.png").href;
```

## Introduction

Within the neuroscience world, the ability to view neurological systems in vivo (living organisms) is crucial to better understanding the functionality and structural features of the human nervous system. This approach is done through neuroimaging techniques such as electrophysiological recordings and magnetic resonance imaging (MRI). Fortunately and unfortunately, these imaging techniques come with many compromises. More specifically, each imaging technique walks a line between optimizing spatial resolution and temporal resolution. In other words, imaging techniques will either provide a really well-defined image of the structure of the nervous system or a well-defined image of the activity of the nervous system throughout time. As of right now, there is no prominent technique that bridges these two resolutions. Hence, the SuperEEG model was born. 

```js
display(html`<figure style="margin: 1rem 0; max-width: 600px;">
  <img src="${imgUrl}" alt="SuperEEG overview" style="width: 100%; height: auto; border-radius: 8px; display: block;">
  <figcaption style="margin-top: 0.5rem; font-size: 0.875rem; color: var(--theme-foreground-muted); font-style: italic;">
    Figure 1: Plot showcaisng different techniques to study the nervous system and what spatial (y-axis) and temporal resolution (x-axis) they relate too. The boxes that are filled in represent perturbation techniques. Boxes that are not filled in represent measurement techniques. (Sejnowski et al, 2014)
  </figcaption>
</figure>`);
```
## Intution 

SuperEEG is a Gaussian process model that leverages multi-subject electrophysiological data to reconstruct neurological activity. The model specifically reconstructes voltage values that would have been picked up empirically if an electrode sensor were to be at a given location. The model performs best when given electrocorticographic data (ECoG). ECoG places electrodes intracranially and deep within brain tissue unlike other counterparts such as electroencephalogram (EEG). This not only provides high-resolution electrical readings (through local field potentials) but also yields structural data through the placement of electrodes. Since ECoG is an invasive technique, the number of electrodes that can be placed within a subject is limited. Hence, the given structural data is normally insufficient for yielding high spatial resolution. The model circumvents this issue by aggregating data from multiple subjects. Instead of looking at only 20-30 electrodes, the model allows for thousands of electrodes to be monitored/reconstructed.

```js
display(html`<figure style="margin: 1rem 0; max-width: 600px;">
  <img src="${pngUrl}" alt="SuperEEG intution" style="width: 100%; height: auto; border-radius: 8px; display: block;">
  <figcaption style="margin-top: 0.5rem; font-size: 0.875rem; color: var(--theme-foreground-muted); font-style: italic;">
    Figure 2: The model uses a Gaussian process, which allows for unknown locations to be correlated with known locations. The model can then leverage that correlation to reconstruct activity. In this figure, patient 1 has recordings in locations A and B, while location C is unknown. Patient 2 has recordings from location B and C, while location A is unknown. The model creates a covariance matrix using all locations from which the correlation between the different locations can be derived. Using the covariance, the model can then reconstruct activity at unknown locations. (Owen et al, 2022)
  </figcaption>
</figure>`);
```

## Methods
Main methdology used to make and test the model is similar to that seen within Lucy L W Owens 



