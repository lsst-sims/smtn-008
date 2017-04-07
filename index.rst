..
  Technote content.

  See https://developer.lsst.io/docs/rst_styleguide.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label
     :target: http://target.link/url

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. Add content below. Do not include the document title.


Introduction
============

Running self-calibration (aka ubercal) on an LSST-sized dataset is difficult (though `not impossible <http://ls.st/doc-15125>`_). It would be convenient if there was an external source of calibration stars we could use to set LSST observation zeropoints. 

GAIA and ULYSSES
================

The GAIA mission will observe around a billion stars. The BP/RP spectrograph will take low resolution spectra of the GAIA targets. `ULYSSES <http://www.mpia.de/gaia/projects/ulysses>`_ can simulate the expected BP/RP observations given a stellar spectrum as input. 

Python wrappers to the ULYSSES code can be found in the `sims_gaia_calib repo <https://github.com/lsst-sims/sims_gaia_calib>`_.

Generating a GAIA-like Catalog
==============================

By default, ULYSSES output is in terms of electrons. To calibrate the output spectra to physical units, we run a flat spectrum source through ULYSSES and use the noiseless output to define the instrument response function. We are essentially assuming GAIA will be able to calibrate it's spectra to a level where the calibration does not significantly contribute to the final spectra noise. With the calibrated ULYSSES spectra, we can compute synthetic LSST magnitudes for the *g,r,i,z,y* filters. The BP spectra do not extend far enough into the blue to compute LSST *u* magnitudes.


.. figure:: /_static/example_input_spec.png
   :name: fig-example_input

   Example stellar spectrum input to ULSYSSES with the LSST filters shown.

.. figure:: /_static/example_output_spec.png
  :name: fig-example_output

  Example of the resulting BP/RP spectra when the input spectrum is passed through ULYSSES.

The BP and RP channels have a region of overlap. For simplicity, we use the BP output blueward of 675 nm and the RP output redward. In theory, one could slightly increase the SNR in the overlap region by properly weighting and combining the spectra. This would only impact the LSST *r* filter.


We use a subset of the Gaia GUMS catalog to generate Gaia end-of-mission (i.e., 75 transits) quality spectra for all the stars down to G~20 in a single LSST pointing. For each star, we assign a suitable Kurucz model SED and use ULYSSES to compute a Gaia specrtum. We then compute synthetic LSST magnitudes for each star. 

.. figure:: /_static/g_resids.png
   :name: fig-g_resids
   :scale: 75

   Residuals of recovered *g* magnitudes.

.. figure:: /_static/r_resids.png
   :name: fig-r_resids
   :scale: 75

   Residuals of recovered *r* magnitudes.

.. figure:: /_static/i_resids.png
   :name: fig-i_resids
   :scale: 75

   Residuals of recovered *i* magnitudes.

.. figure:: /_static/z_resids.png
   :name: fig-z_resids
   :scale: 75

   Residuals of recovered *z* magnitudes.

.. figure:: /_static/y_resids.png
   :name: fig-y_resids
   :scale: 75

   Residuals of recovered *y* magnitudes.



Results
=======

Note the GUMS field which we used is located at (RA, dec) = (340.104, 27.547) with 33,000 stars (13,000 in the range 17 < g <  19). Scaling to the galactic pole, we would expect the density of stars to drop to ~25% that level. So we would still have ~30 Gaia stars per LSST CCD at the galactic pole that could be used for calibration. 



Recovering the u-band
=====================

The synthetic y-band magnitudes are still usable because the LSST y throughput is very low in the region where Gaia cuts off. That is not true for the u-band, thus, if we are going to use Gaia to calibrate the u filter, there needs to be an extra step in extrapolating Gaia observations to LSST u-magnitudes.

One possible solution is to use Gaia derived stellar parameters (Teff, Fe/H, log g) along with Kurucz models to interpolate the expected LSST *u* magnitude. `Lui et al <http://adsabs.harvard.edu/abs/2012MNRAS.426.2463L>`_ look at how well Gaia will be able to recover stellar parameters. 


.. figure:: /_static/kuruz_met.png
   :name: fig-kurucz-met

   Kurucz model grid.

.. figure:: /_static/kuruz_logg.png
   :name: fig-kurucz-logg

   Same as :numref:`fig-kurucz-met`, but color-coded by stellar log g.

.. figure:: /_static/u_perfect.png
   :name: fig-u-perfect

   If we assume Gaia returns perfect stellar parameters, the Gaia synthetic LSST *g* and *r* magnitudes can be used with
   the Kurucz models to generate LSST *u* magnitudes with the plotted residual distribution. Results in 0.005 mag RMS at u=18.


.. figure:: /_static/u_good.png
   :name: fig-u-good

   Same as :numref:`fig-u-perfect`, but inserting 0.1 dex RMS errors on both the metallicity and log g Gaia values.  Results in 0.002 mag RMS at u=18.


.. figure:: /_static/u_poor.png
   :name: fig-u-poor

   Same as :numref:`fig-u-perfect`, but inserting 0.35 dex RMS errors on the metallicity and 0.2 dex errors on log g. Results in 0.06 mag RMS at u=18.



It should be possible to construct a u-band stellar catalog from the Gaia data that would be adequate for calibrating LSST observations **if**

* stars can be described by Kurucz models

* Gaia returns stellar parameters with their expected precision


As a check on how well Kurucz models can convert Gaia observations into *u-g* colors, we take the `stsci grid <http://www.stsci.edu/science/starburst/Kurucz.html>`_ of models (plotted in :numref:`fig-kurucz-met`) and withhold a random 10% of the points (110 points) and use the remaining 90% (990 points) to interpolate the expected $u-g$ color using the scipy LinearNDInterpolator which uses Qhull and rescales the input dimensions.  

For red stars (*u-g* > 0.5), the *u-g* color of the interpolated points has an RMS error of 0.04 mag. It may be possible to reduce the interpolation error by using a finer grid of stellar atmospheres, or possibly using a more sophisticated interpolation method. 

.. figure:: /_static/interp_verify.png
   :name: fig-interp_verify

   Testing the ability to correctly interpolate u-g color from Kurucz models given g-r, metallicity, and log g. 



Other Issues
============

Besides the difficulty in extrapolating the u-band, Gaia will not observe as deep in the galactic plane. This leaves the possibility that there will not be any overlap in the Gaia observations and LSST stars that are not saturated. 

The Gaia `data release scenarios <https://www.cosmos.esa.int/web/gaia/release>`_ do not include releasing the reduced BP/RP spectra, but only the derived stellar parameters. Thus we may need to request the Gaia collaboration compute synthetic LSST magnitudes or expand the scope of their data releases to include BP/RP (non-integrated) spectra.



