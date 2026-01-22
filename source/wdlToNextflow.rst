WDL to Nextflow
===============

.. contents::
   :depth: 2
   :local:
   :backlinks: top

Workflow general structure
--------------------------
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
----------------------------------
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
-----------
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
     - Default value
     - ::

        ~{ default="default_value" var }
     - ::

        ${ var ?: "default_value" }
   * - 6.
     - Glob in result
     - ::

        glob("output/file_*.txt")
     - ::

        "output/file_*.txt"
   * - 7.
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
   * - 8.
     - Array element access
     - ::

        x = array[i]
     - ::

        //Use Groovy function, array can be channel
        x = get( array, i )

Task elements
-------------

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
   * - 3.
     - Private declarations
     - ::

        task task1 {
        …
        <Type> variable = <Value>
        …
        command { ...

     -  ::

         process task_name
         …
         script
         variable = <Value>
         """
         …
   * - 4.
     - Runtime options
     - ::

        runtime {
           maxRetries: 3
           memory: "N GB"
           preemptible: 1
           disks: "local-disk ~{disk_size} HDD"
           cpu: threads
           docker: "image name"
       }
     -  ::

         
         maxRetries 3
         memory "4*4 GB"

  
         cpus 4
         container 'image_name'
   * - 5.
     - Optional inputs
     - ::

        input {
           Sting mandatory
           String opt = “Default”
        }
     - ::

        input:
        val mandatory
        val opt
        …
        script:
        opt = getDefault(opt, “Default”)
        """
        …
        """

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
   * - 2.
     - Calling task with optional inputs
     - ::

        call task1 { 
           mandatory = value
           #optional = value2
        }
     - ::

        task1( value , “NO_VALUE”)
   * - 3.
     - Multiple calls of task defined in the same document
     - ::

        task task1 {…}

        call task1 as t1 { }
        call task1 as t2 { }
     - Not supported
   * - 4.
     - Expressions inside if
     - ::

        if (x > 2) {
           Int y = x*2
        }
     - ::

        y = x.map{ (it>2)? it*2 : null }
Cycles
------

.. list-table::
   :header-rows: 1
   :widths: 20 80 80 80

   * - #
     - Description
     - WDL
     - Nextflow
   * - 1.
     - Simple cycle
     - ::

        scatter ( file in files ) {
           call task1 { input: arg1 = file }
        }
     - ::

        file = files
        task1( file )
   * - 2.
     - Cycle in range
     - ::

        scatter ( i in range(length(С)) ) {
           call task1 { 
              input: arg1 = i }
        } 
     - ::

        //Use Groove functions
        i = range( length(С) )
        task1( i )
   * - 3.
     - Expressions in cycle
     - ::

        scatter ( i in array ) {
          Int j = i*2
          Int k = i*3
        }
     - ::

        j = toChannel( array ).map { i -> i*2 }
        k = toChannel( array ).map { i -> i*3 }
   * - 4.
     - Nested cycles
     - ::

        scatter ( i in array ) {
           scatter ( j in array2 ) {
              Array[Int] pair = [ i,j ]
           }
        }
     - ::

        pair = toChannel( array ).combine(toChannel(array2)).map{ i,j -> [i,j] }

Supplementary functions 
-----------------------

.. list-table::
   :header-rows: 1
   :widths: 20 80 80 80

   * - #
     - Function
     - Description
     - Code
   * - 1.
     - get(arr, i)
     - Returns  element of arr with index i, arr may be Array or Channel.
     - TBA
   * - 2.
     - getDefault(var, Default)
     - Returns var or Default if var is not defined.
     - TBA
   * - 3.
     - toChannel(arr)
     - Creates channel from arr, if arr is channel already does nothing.
     - TBA
 
 
WDL functions implemented in Groovy 
-----------------------------------

.. list-table::
   :header-rows: 1
   :widths: 20 80 80

   * - #
     - Function
     - TBA
   * - 1.
     - basename(path)
     - TBA  
   * - 2.
     - sub(input, pattern, replacement)
     - TBA  
   * - 3.
     - ceil
     - TBA  
   * - 4.
     - length
     - TBA  
   * - 5.
     - range
     - TBA  
   * - 6.
     - select_first
     - TBA  
   * - 7.
     - select_all
     - TBA  
   * - 8.
     - defined
     - TBA  
   * - 9.
     - read_string
     - TBA  
   * - 10.
     - read_int
     - TBA  
   * - 11.
     - read_float
     - TBA  

Not supported yet 
-----------------

.. list-table::
   :header-rows: 1
   :widths: 20 80 80

   * - #
     - Name
     - Description
   * - 1.
     - Comments
     - Comments.

