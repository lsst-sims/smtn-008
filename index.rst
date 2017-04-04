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

.. note::
   **Writing and experiments in progress**

Introduction
============

Running self-calibration (aka ubercal) on an LSST-sized dataset is difficult (though not impossible xxx-cite self cal doc). It would be convenient if there was an external source of calibration stars we could use to set LSST observation zeropoints. 

GAIA and ULYSSES
================

The GAIA mission will observe around a billion stars. The BP/RP spectrograph will take low resolution spectra of the GAIA targets. `ULYSSES <http://www.mpia.de/gaia/projects/ulysses>`_.

Python wrappers to the ULYSSES code can be found in the `sims_gaia_calib repo <https://github.com/lsst-sims/sims_gaia_calib>`_.

Generating a GAIA-like Catalog
==============================

By default, ULYSSES output is in terms of electrons. To calibrate the output spectra to physical units, we run a flat spectrum source through ULYSSES and use the noiseless output to define the instrument response function. We are essentially assuming GAIA will be able to calibrate it's spectra to a level where the calibration does not significantly contribute to the final spectra noise. 

xxx-figures of input spectrum, output from ULYSSES, post response function.

.. figure:: /_static/example_input_spec.png
   :name: fig-example_input

   Example stellar spectrum input to ULSYSSES with the LSST filters overplotted.

.. figure:: /_static/example_output_spec.png
  :name: fig-example_output

  Example of the resulting BP/RP spectra when the input spectrum is passed through ULYSSES.

The BP and RP channels have a region of overlap. For simplicity, we use the BP output blueward of 675 nm and the RP output redward. In theory, one could slightly increase the SNR in the overlap region by properly weighting and combining the spectra. This would only impact the LSST *r* filter.


We use a subset of the Gaia GUMS catalog to generate Gaia end-of-mission quality spectra for all the stars down to G~20 in a single LSST pointing. We then compute synthetic LSST magnitudes for each star. 

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


The GAIA wavelength coverage does not extend to cover all the *u* filter.  We thus define new *u_short* filter that is identical to the LSST filter, but has sharp cutoffs so it remains in the Gaia wavelength coverage.


.. figure:: /_static/ u_truncated_resids.png
   :name: fig-u_resids
   :scale: 75

   Residuals of recovered *u*-truncated magnitudes.


Results
=======

Note the GUMS field which we used is located at (RA, dec) = (340.104, 27.547) with 33,000 stars (13,000 in the range 17 < g <  19). Scaling to the galactic pole, we would expect the density of stars to drop to ~25% that level. So we would still have ~30 Gaia stars per LSST CCD at the galactic pole. 



Recovering the u-band
=====================

The synthetic y-band magnitudes are still useable because the LSST y throughput is very low in the region where Gaia cuts off. That is not true for the u-band, thus, if we are going to use Gaia to calibrate the u filter, there needs to be an extra step in extrapolating Gaia observations to LSST u-magnitudes.

Two possible methods:
1) Because there is some overlap between Gaia BP specta and LSST u, one could use model spectra to construct a synthetic u-u_gaia v u_gaia-g diagram from model spectra, then recover u from the Gaia data. 
2) Gaia claims to return full stellar parameters for every star (Teff, Fe/H, log g). If those parameters are acccurate and precice enough, they could be converted to a model stellar spectrum and the LSST u could be computed. There is a risk of making things slightly circular, using GAIA derived stellar parameters to infer LSST u-magnitudes, which are then used to compute LSST colors that are used to fit stellar parameters. 


Other Issues
============

Besides the difficulty in the u-band, Gaia will not observe as deep in the galactic plane. This leaves the possibility that there will not be any overlap in the Gaia observations and LSST stars that are not saturated. 


Notes to self
=============

Things I need to do:
* work out u_short to u conversion
* Need to make an LSST catalog of the stars, with errors



.. note::

   **This technote is not yet published.**

