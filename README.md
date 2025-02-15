pfSense DPDK and VPP implementation:
------------------------------------
This is a repository to add DPDK and VPP support to pfSense.

Installation:
-------------
You can find pre compiled packages build for pfSense CE 2.7.2 inside the pkg directory.
To install DPDK and VPP on your pfSense CE 2.7.2, you have to download all the packages and run :
pkg add dpdk22.11-22.11.2.1400094_1.pkg
pkg add vpp-24.06_1.pkg

To build (for other pfSense versions / kernel):
-----------------------------------------------
See BUILD_fr.md (for the moment in french) for instruction on how to build those packages
