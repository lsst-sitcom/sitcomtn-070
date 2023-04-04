:tocdepth: 1

.. sectnum::

Abstract
========

Determining the length of exposures needed when using the Collimated Beam Projector

Overview
========

Want to determine the exposure time required by the CBP, being fed by a tunable laser, for a given SNR. 

.. math:: \textrm{exptime} = \frac{SNR^{2}}{\textrm{ph_rate}}

Of the following diagram, we will cover the Laser+Fiber lightsource, CBP and LSST boxes. Other elements of the calibration ETC can be found in the `Flatfield Calibration ETC Tech Note <https://sitcomtn-049.lsst.io>`__.

.. figure:: /_static/etc_overview.png
   :name: etc_overview
   :target: ../_images/etc_overview.png
   :alt: etc_overview

   ETC Overview

Laser Input
===========
Parameters:
 - ``laser_file``: filename that contains the Watts/nm from the laser 
 - ``decrease_expected``: This is the percentage decrease from the values in the ``laser_file``. This should be zero if the ``laser_files`` contains measured levels.  

We are using the Ekspla NT242 currently. It is possible that we may have a backup laser with a different output profile. Currently, we are using :file:`PGD151_NT242.txt`, which is the expected levels from the distributor. We will want to update this with measured values.

.. figure:: /_static/pgd151_nt242_output.png
   :name: pgd151_nt242_output
   :target: ../_images/pgd151_nt242_output.png
   :alt: pgd151_nt242_output

   Output of NT242 laser

Fiber Attentuation
------------------
Parameters:
 - ``fiber_length``: Length of fiber from the laser to either the CBP or the flatfield projector. 
 - ``fiber_type``: This is the type of ceramoptic fiber we expect to use. 
 - ``fiber coupling``: This is the throughput decrease based on the coupling between the fiber and the laser. 
 - ``use_fiber``: Whether or not a fiber will be used [True]

Based loosely on LTS-664, I estimate that the fiber will run ~8m from the laser to the CBP. 

Likely, will get this fiber from ceramoptic: https://www.ceramoptec.com/products/fibers/optran-uv-/-wf.html.
The attenuation (dB/km) for several kinds of fibers was sent to me by Ceramoptic (email) and ``WFNS`` was recommended.

.. figure:: /_static/ceramoptic_attenuation.png 
   :name: ceramoptic_attenuation
   :target: ../_images/ceramoptic_attenuation.png 
   :alt: ceramoptic_attenuation 

   Attenutation of ceramoptic UV/WS fibers.

Tranmission of the fiber is then calculated:

.. math:: \textrm{T} = 10^{\frac{-dB/km}{distance(km)/10}}

Based on initial measurements, the ``fiber_coupling`` is estimated to be 0.5 across all wavelengths.


CBP Throughput
==============

Integrating Sphere
------------------
Parameters:
 - ``sphere_reflectance`` [:math:`\rho`]: Reflectivity of integrating sphere 
 - ``exit_port_diameter``: Size of the exit port of the integrating sphere (inch)
 - ``port_diameters``: List of the port diameters on the integrating sphere, including the exit port (list in inches)
 - ``sphere_diameter``[:math:`D_{sphere}`]: Diameter of the integrating sphere (inch)

Currently using  3P-GPS-060-SF AS-02266-060 from Labsphere, which uses Spectraflect coating on a 6 inch diameter sphere with and exit pupil of 2.5 inches and two 1 inch ports, one used for input of the fiber and the other for the photodiode. I am assuming a mean reflectance values of 0.985 for the Spectraflect.

The following calculations are taken from here the `Labsphere Guide to Integrating Spheres <https://www.labsphere.com/wp-content/uploads/2021/09/Integrating-Sphere-Theory-and-Applications.pdf>`__.

Calculate the surface area of the inside of the ingrating sphere after having converted the :math:`D_{sphere}` to radius in m from inches (:math:`r_{sphere}`)

.. math:: A_{sphere} = 4 \cdot \pi \cdot r_{sphere}^{2}

Calculate the total area of the ports in the integrating sphere, where :math:`r_{i}` is the radius of each port).

.. math:: f = \sum_{i}^{\textrm{ports}} \pi \cdot r_{i}^{2}

Finally, calculate the intensity of light exiting the integrating sphere, by multiplying Ls by the flux incident on the integrating sphere.

.. math:: \textrm{Ls} = \frac{1}{\pi \cdot A_{sphere}} \times \frac{\rho}{1-\rho \cdot (1-f)}


Mask Efficiency
---------------
Parameters:
 - ``pinhole_size``: Size of the mask pinhole [m]
 - ``distance_to_mask``: Distance from the exit pupil of integrating sphere and mask [inches] 

There are a variety of mask designs that are being considered with a range of pinhole sizes. This ETC is being used, in part, to evaluate different mask designs.

The current design has the ``distance_to_mask`` at approximately 3 inches.

When then need to determine how much light from the integrating sphere is incident on the mask.

Measure the angle from the exit pupil (:math:`r_{exit}`) to the mask

.. math:: \theta = arctan(\frac{r_{exit}}{d_{mask}})

Then calculate the solid angle of light making it to the maks

.. math:: SA = \pi \cdot sin(\theta)^{2}

Finally, multiply by the area of the mask, calculated as :math:`A_{mask} = \pi \cdot r_{mask}^{2}` to the get the mask efficiency. 

.. math:: \epsilon_{mask} = A_{mask} \cdot SA


CBP Efficiency
--------------
Parameters:
 - ``cbp_tranmission``: Tranmission of CBP optics. 
 - ``f_num_cbp``: f/# of the CBP [2.63]
 - ``f_cbp``: Focal length of the CBP (m) [0.635]
 
The transmission of the CBP optics was measured by the vendor to be 0.55. We estimate that it is now closer to 0.5

First measure how much light from the mask is getting into the CBP:

.. math:: P = \frac{\pi}{(2 \cdot \textrm{f/#}_{CBP})^{2}}

And then multiply this by the overall transmission of the CBP optics.

Telescope and Camera Throughput
===============================
Parameters:
 - ``total_number_of_pixels``: 3.2e9
 - ``pixel_size``: 10e-6 m
 - ``f_lsst``: focal length of the LSST telescope (m) [10.3]

Mirror Reflectance
------------------
Parameters:
 - ``m1``, ``m2``, ``m3``: Reflectance for a mirror coating; options:[``Al-Ideal``, ``Al-Aged``, ``Al-Ag``]

There are three mirrors [m1, m2, m3] that will be coated with either Al or Ag. The full throughput will be the combination of the three mirrors, whether all have the same coating or different. The curves we are using for the Al coatings come from the `Baseline Design Throughput <https://docushare.lsst.org/docushare/dsweb/View/Collection-1777>`__ on Docushare. The Al-Ag coating can be found on the `Mirror Coating Recipe <https://docushare.lsst.org/docushare/dsweb/View/Collection-1047>`__ on Docushare.

.. figure:: /_static/mirror_coating_reflectance.png
   :name: mirror_coating_reflectance
   :target: ../_images/mirror_coating_reflectance.png
   :alt: mirror_coating_reflectance

   Reflectance of telescope mirror coatings

Filter Throughput
-----------------
Using the filter throughput from the `Baseline Design Throughput <https://docushare.lsst.org/docushare/dsweb/View/Collection-1777>`__ on Docushare.

.. figure:: /_static/ideal_filters.png
   :name: ideal_filters
   :target: ../_images/ideal_filters.png
   :alt: ideal_filters

   Ideal filter throughput

Detector Efficiency
-------------------
Parameters:
 - ``detector_file``: File with QE for the detector 

Currently using the QE curve for the e2v detectors (:file:`detector_e2vPrototype.dat`) from the `Baseline Design Throughput <https://docushare.lsst.org/docushare/dsweb/View/Collection-1777>`__ on Docushare.

.. figure:: /_static/detector_e2v_qe.png
   :name: detector_e2v_qe
   :target: ../_images/detector_e2v_qe.png
   :alt: detector_e2v_qe

   QE for e2v detectors

Readout Overheads
=================
Parameters:
 - ``cam_readout``: readout time for LSSTCam [2 sec.]
 - ``min_exptime``: The minimum exposure time allowed by the camera [15 sec.] 
 - ``electrometer_readout``: The readout time for the electrometer [not currently set]
 - ``spectrograph_readout``: The readout time for the spectrograph [not currently set]

The exposure time overheads are quite simplistically calculated at this time. Essentially, we can only take an exposure every 17 seconds. Therefore, if we require less than that time to reach the required SNR, the total exposure time is 15 seconds plus an additional 2 seconds of readout time. If we require more than 15 seconds of exposure to reach teh required SNR, we will add additional exposures of length 15 seconds until it is met. Each 15 second exposure requires a 2 second readout time.

I am not currently calculating the readout time required for the electrometer. This will have to be addressed very soon. 

Exposure Time Calculator
========================
The exposure time calculator is saved in :file:`rubin_calib_etc.py` and runs given a configuration file, like :file:`calib_etc.yaml`. 

First, photons per pixel are calculated, by taking the following steps:

1. Calculate irradiance from laser + fiber into the CBP integrating sphere

2. Multiply by the CBP transmission, which includes the integrating sphere, mask efficiency and cbp throughput to get irradiance on telescope

3. Calculate number of photons hitting telescope

.. math:: \textrm{photon_rate} = Watts \times \frac{\lambda(m)}{(h \cdot c)}

4. Multiply by the telescope, filter and camera efficiency curves

5. Divide total photons detected by total number of pixels

6. Finally, Then the size of the spot is calculate for a final SNR per spot:

.. math:: M = f_{lsst}/f_{cbp}

.. math:: D_{spot} = \frac{(\textrm{pinhole_size} \cdot \textrm{M})}{\textrm{pixel_size}}

.. math:: \textrm{spot_total_pixels} = \pi \cdot (D_{spot}/2)^{2}



Sample Results
==============

Using a 6 inch integrating sphere with a mask with a pinhole size of 150um and all telescope mirrors being ``Al-Ag``, you would get the following photon rate and total exposure times:

.. figure:: /_static/photon_rate_150um_cbp.png
   :name: photon_rate_150um_cbp
   :target: ../_images/photon_rate_150um_cbp.png
   :alt: photon_rate_150um_cbp

   Photon rate per spot for CBP with 150um pinhole



.. figure:: /_static/exptime_150um_cbp.png
   :name: exptime_150um_cbp
   :target: ../_images/exptime_150um_cbp.png
   :alt: exptime_150um_cbp

   Total exposure times for SNR = 300 for each spot


.. Make in-text citations with: :cite:`bibkey`.
.. Uncomment to use citations
.. .. rubric:: References
.. 
.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
