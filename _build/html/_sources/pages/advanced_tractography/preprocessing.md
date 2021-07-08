# Preprocessing
_by Lawrence Binding_

```{admonition} Estimated Time 
:class: TIP
30 minutes
```

So MRtrix3 is run through lines of code, this can be slightly daunting at first but allows us more freedom when running tractography on large datasets. If you are unfamiliar with how to use the terminal (Mac) or MSYS2 (Windows) then please see the additional guides section for a quick how-to. 

The first thing we need to do with our diffusion data is to clean it. Diffusion data is messy, very messy and the signal to noise radio is small, very small. The best way we can improve this is by using various correction methods on our diffusion data. 
<style>
h1 {text-align: center;}
h2 {text-align: center;}
</style>

--- 

## Converting to MRtrix3 Format
To begin with, we need to get the data in MRtrix3 format (.mif) so we can use the denoising tools. So in the advanced_tractography folder we have our fibrecup.nii.gz and our gradient file (which contains the bvecs and bvals), the question is how do we put them all into one file which MRtrix3 can use. We are able to do this using mrconvert:

```shell
mrconvert fibrecup.nii.gz -fslgrad bvecs bvals fibrecup.mif 
```
The above command tells our terminal to use MRtrix3 mrconvert command to take our fibrecup.nii.gz, import the gradient file, and output them in MRtrix3 format .mif. the MIF format contains the bvecs and bvals in the header information, which means less can go wrong!

---
## Viewing our Image  
Now we've got our diffusion in MRtrix3 format, lets view our image using the below command:

```shell
mrview  fibrecup.mif
```
Here we are telling mrview (MRtrix3 viewing software) to open the next file

---
## Denoising  
So lets move onto preprocessing the data, first we should denoise our DWI image, this removes thermal noise in the image using the Marchenko-Pastur PCA method. This must be done before any other types of processing is undertaken: 


```shell
dwidenoise fibrecup.mif fibrecup_denoise.mif -noise noise.mif
```
To check our results we need to calculate the difference between the raw and denoised images:

```shell
mrcalc fibrecup.mif fibrecup_denoise.mif -subtract denoise_difference.mif 
mrview noise.mif denoise_difference.mif
```

<figure>
<img src="../../_static/img/dwidenoise.png" alt="colab" style="width:396px;height:239px;">
<figcaption>Fig.1 - Before (left) and after (right) using dwidenoise.</figcaption>
</figure>

---
## Gibbs Ringing Correction
Next we need to remove Gibb's ringing artifacts. These occur due to limitations in the fourier transformation. 

```shell
mrdegibbs fibrecup_denoise.mif fibrecup_denoise_gibbs.mif
```

To check our results, we can culate the difference and inspect them. 

```shell
mrcalc fibrecup_denoise.mif fibrecup_denoise_gibbs.mif -subtract residual.mif
mrview fibrecup_denoise_gibbs.mif residual.mif
```

<figure>
<img src="../../_static/img/ringing.png" alt="colab" style="width:363px;height:336px;">
<figcaption>Fig.2 - Before (left) and after (right) using mrdegibbs.</figcaption>
</figure>

--- 

## Distortion correction [EDDY]
There are several different methods of correcting for distortions induced by motion and echo planar imaging distortions all with different advantages. Synb0-DISCO is a newer method which doesn't require any additional acquisitions in scanner not only saving time but retrospective data. However, this is run in Docker and we don't have time to tell you about this. Instead we're going to be using a tool called EDDY which is packaged in MRtrix3 as dwipreproc. 

```shell
dwipreproc fibrecup_denoise_gibbs.mif fibrecup_denoise_gibbs_preproc.mif -rpe_none -pe_dir ap
```

<figure>
<img src="../../_static/img/eddy.png" alt="colab" style="width:435px;height:278px;">
<figcaption>Fig.3 - Before (left) and after (right) using dwipreproc.</figcaption>
</figure>

## Field Bias Correction 
To improve brain mask estimation, we need to run a bias field correction which will normalise the image. 

```shell
dwibiascorrect ants fibrecup_denoise_gibbs_preproc.mif fibrecup_denoise_gibbs_preproc_biaCorr.mif -bias bias.mif 
```
To check the bias field we just load that into mrview.

```shell
mrview bias.mif -colourmap 2  
```
<figure>
<img src="../../_static/img/biascorrect.png" alt="colab" style="width:540px;height:201px;">
<figcaption>Fig.4 - Before (left) and after (right) using dwibiascorrect.</figcaption>
</figure>


## Conclusion 
There are a lot of steps required to ensure your diffusion data is the best it can be. Here we have covered the most advanced and readily available in MRtrix3. From this you will now confidently be able to analyse your diffusion data!


<style>
  .iframe-container {
		text-align:center;
  		width:100%;
  }
</style>


```{admonition} Further reading
- blah blah 
- blah blah
- blah blah
- blah blah

```
