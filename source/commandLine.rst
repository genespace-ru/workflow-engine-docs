Command line
============
Requirements
------------
You will need Java 21 (jdk or jre) to run this tool.

Download
--------

Download executable jar WDL2Nextflow.jar at https://github.com/genespace-ru/workflow-engine.

Usage
-----

- To convert WDL to Nextflow:

.. code-block::

  java -jar WDL2Nextflow.jar <PATH_TO_WDL> 

Where <PATH_TO_WDL> is path to the WDL file. It will create file with the same name as <PATH_TO_WDL> with extension .nf.

- To generate visual diagram:
.. code-block::

  java -jar WDL2Nextflow.jar <PATH_TO_WDL> -i

Where <PATH_TO_WDL> is path to the WDL file. It will create image file with the same name as <PATH_TO_WDL> with extension .png.
