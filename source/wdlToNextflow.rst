WDL to Nextflow
===============


Workflow general structure
-------------
.. list-table::
   :header-rows: 1
   :widths: 100 100

   * - WDL
     - Nextflow
   * - ::

        version <VERSION>
        #Processes imported from other diagrams
        import “diagram_1” as ext_1
        …

        #Processes (or tasks) defined in diagram
        task task_1 { 
          ##Task body
        }
        …

        #Main workflow
        workflow mainWorkflow {
   
           input {
             <Type> p_1
             …
           }

           #Workflow body
           call task_1 as call_1 { input_1 = value_1, … }
            … 
           expression_1 
           …
           scatter_1 (element in array) {
             #Scatter body
           }
           …

           output{
             <Type> p_1 = <Expression>
             …
           }
        }
     - ::

        //Groovy functions defined in special nextflow document
        include { basename; sub; length; ... } from './biouml_function.nf'

        //Processes imported from other diagrams
        include { ext_process_1 as ext_1 } from './diagram_1'
        …

        //Default Global workflow inputs
        params.p_1
        …

        //Processes defined in diagram
        process_1 {…}
        …

        //Main workflow
        workflow mainWorkflow {
          take:
          p_1
          …

          main:
          call_1 {…} 
          expression_1
          …

          emit:
          result_1 = expression_1
          …
        }

        //Entry point workflow
        workflow {
           mainWorkflow( params.p_1, …, params.p_k )
        }

Task and Process general structure
-------------
.. list-table::
   :header-rows: 1
   :widths: 100 100

   * - WDL
     - Nextflow
   * - ::

        task task_name {
           
           input {
             <Type> input_1
             …
             <Type> optional_input_1 = <Expression>
             …
           }
            
           <Type> variable_1 = <Expression>
            …
        
           command {
              <Executable script>
           }

           output {
             <Type> output_1 = <Expression>
           }

           runtime {
             <Runtime properties>
           }
        }

     - ::

        process p_1 {

           input :
           <Type> input_1
            …
 
           publishDir "p_1", mode: 'copy'  
           fair true  

           script:
           variable_1 = <Expression>
            …
           """
           <Executable script>
           """

           output:
           path output_1, emit: out_1
           …
        }


Type mapping
------------
.. list-table::
   :header-rows: 1
   :widths: 20 80 80

   * - #
     - WDL
     - Nextflow
   * - 1.
     - File, Array[File]
     - path
   * - 2.
     - Boolean, String, Integer, Float, Array[Boolean], Array[String], Array[Integer], Array[Float]
     - val
   * - 3.
     - Map, Struct, Object
     - val

Imports 
-------
.. list-table::
   :header-rows: 1
   :widths: 20 80 80 80

   * - #
     - Description
     - WDL
     - Nextflow
   * - 1.
     - Import task from other script
     - ::

        import “script” as ext
        ...
        call ext.task1 as t { … }
     - ::

        include { task1 as t } from './script
        ...
        t( … )
   * - 2.
     - Call imported task twice
     - ::

        import “script” as ext
        …
        call ext.task1 as t1 { … }
        call ext.task1 as t2 { … }
     - ::
        
        include { task1 as t1 } from './script
        include { task1 as t2 } from './script
        …
        t1( … )
        t2( … )
   * - 3.
     - Import external workflow
     - ::

        import “script” as ext
        call ext.mainWorkflow  as alias { … }

     - ::
        
        include { mainWorkflow as alias } from './script

Expressions
-----------------
.. list-table::
   :header-rows: 1
   :widths: 20 80 80 80

   * - #
     - Description
     - WDL
     - Nextflow
   * - 1.
     - Workflow variable
     - ::

        ~{var_name}
     - ::

        ${var_name}
   * - 2.
     - Bash variable
     - ::

        ${var_name}
     - ::

        \${var_name}
   * - 3.
     - Cycle in script
     - ::

        for file in ~{sep=" " files}
     - ::

        for file in ${ files.join() }
   * - 4.
     - Ternary operator
     - ::

        ~{ if ... then ... else ... }
           if ... then ... else ...
     - ::

        ... ? ... : ...
   * - 5.
     - Glob in result
     - ::

        glob("output/file_*.txt")
     - ::

        "output/file_*.txt"
   * - 6.
     - Predefined functions
     - ::

        basename( file )
        sub( input, pattern, replacement )
        length( array )
        ...
     - ::

        include { basename; sub; length; ... } from './biouml_function.nf'
        basename( file )
        sub( input, pattern, replacement )
        length(array)
        ...


Tasks
-----
.. list-table::
   :header-rows: 1
   :widths: 20 80 80 80

   * - #
     - Description
     - WDL
     - Nextflow
   * - 1.
     - Task metadata
     - ::

        meta {
           description: "text"
        }
     -  Not supported
   * - 2.
     - Parameters metadata
     - ::

        parameter_meta {
           input_vcf: { 
              help: “help text”
              description: “text”
              suggestions: [“v1”,”v2”]
           }
        }

     -  Not supported




Workflow
--------
.. list-table::
   :header-rows: 1
   :widths: 20 80 80 80

   * - #
     - Description
     - WDL
     - Nextflow
   * - 1.
     - Two consequent steps
     - ::

        call task1 as call1 {
           input:  arg1 = val1 
        }

        call task2 as call2 {
           input: arg1 = call1.result, 
                  arg2 = val2 
        }
     - ::

        result_task1 = task1( val1 )
        task2( result_task1.result, val2 )

