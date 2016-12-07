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

Running self-calibration on an LSST-sized dataset is difficult (though not impossible xxx-cite self cal doc). It would be convenient if there was an external source of calibration stars we could use to set LSST observation zeropoints. 

GAIA and ULYSSES
================

The GAIA mission will observe around a billion stars. The BP/RP spectrograph will take low resolution spectra of the GAIA targets. `ULYSSES <http://www.mpia.de/gaia/projects/ulysses>`_.

Python wrappers to the ULYSSES code can be found in the `sims_gaia_calib repo <https://github.com/lsst-sims/sims_gaia_calib>`_.

Generating a GAIA-like Catalog
==============================

By default, ULYSSES output is in terms of electrons. To calibrate the output spectra to physical units, we run a flat spectrum source through ULYSSES and use the noiseless output to define the instrument response function. We are essentially assuming GAIA will be able to calibrate it's spectra to a level where the calibration does not significantly contribute to the final spectra. 

xxx-figures of input spectrum, output from ULYSSES, post response function.

The BP and RP channels have a region of overlap. For simplicity, we take just the BP output blueward of 675 nm and the RP output redward. In theory, one could slightly increase the SNR in the overlap region by properly weighting and combining the spectra. This would only impact the LSST *r* filter.

The GAIA wavelength coverage does not extend to cover all the *u* filter. The SNR can also get very low in the *y* band. We thus define new *u_short* and *y_short* filters that are identical to the LSST filters, but have sharp cutoffs so they remain in the GAIA wavelength coverage.

xxx-figure of used passbands. 

Notes to self
=============

Things I need to do:
* Verify that GAIA will get BP/RP for just about everything down to g=20
* Run on GUMS stars on a full pointing sized field, so I can check corner chip precision
* Grab stars from fatboy and make a full pointing field at the south galactic pole
* work out u_short to u conversion
* Need to make an LSST catalog of the stars, with errors
* Then, difference of LSST observed mag with GAIA mag, calc uncertainty, compute zeropoint and error on each chip.
* double check that my noise looks reasonable by plugging in stars that are on the example webpage.



.. note::

   **This technote is not yet published.**

   Checking if using calibration stars from GAIA is adequate to replace an ubercal procedure.
