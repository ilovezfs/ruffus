.. include:: ../global.inc
.. _decorators.combinations_with_replacement:
.. index::
    pair: @combinations_with_replacement; Syntax

.. seealso::

    * :ref:`@combinations_with_replacement <new_manual.combinations_with_replacement>` in the **Ruffus** Manual
    * :ref:`Decorators <decorators>` for more decorators


.. |input| replace:: `input`
.. _input: `decorators.combinations_with_replacement.input`_
.. |tuple_size| replace:: `tuple_size`
.. _tuple_size: `decorators.combinations_with_replacement.tuple_size`_
.. |filter| replace:: `filter`
.. _filter: `decorators.combinations_with_replacement.filter`_
.. |extras| replace:: `extras`
.. _extras: `decorators.combinations_with_replacement.extras`_
.. |output| replace:: `output`
.. _output: `decorators.combinations_with_replacement.output`_
.. |matching_formatter| replace:: `matching_formatter`
.. _matching_formatter: `decorators.combinations_with_replacement.matching_formatter`_

################################################################################################################################################
combinations_with_replacement
################################################################################################################################################

************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************
@combinations_with_replacement( |input|_, |filter|_, |tuple_size|_, |output|_, [|extras|_,...] )
************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************

    **Purpose:**

        Generates the **combinations_with_replacement**, between all the elements of a set of |input|_ (e.g. **A B C D**),
        i.e. r-length tuples of |input|_ elements with no repeated elements (**A A**)
        and where order of the tuples is irrelevant (either **A B** or  **B A**, not both).

        The effect is analogous to the python `itertools  <http://docs.python.org/2/library/itertools.html#itertools.combinations_with_replacement>`__
        function of the same name:

            .. code-block:: pycon
                :emphasize-lines: 2-3

                >>> from itertools import combinations_with_replacement
                >>> # combinations_with_replacement('ABCD', 2)
                >>> #   --> AA AB AC AD BB BC BD CC CD DD
                >>> [ "".join(a) for a in combinations_with_replacement('ABCD', 2)]
                ['AA', 'AB', 'AC', 'AD', 'BB', 'BC', 'BD', 'CC', 'CD', 'DD']

        Only out of date tasks (comparing input and output files) will be run

        |output|_ file names and strings in the extra parameters
        are generated by string replacement via the :ref:`formatter()<decorators.formatter>` filter
        from the |input|_. This can be, for example, a list of file names or the
        |output|_ of up stream tasks.
        .
        The replacement strings require an extra level of nesting to refer to
        parsed components.

            #. The first level refers to which *set* in each tuple of |input|_.
            #. The second level refers to which |input|_ file in any particular *set* of |input|_.

        This will be clear in the following example:

    **Example**:

        If |input|_ is four pairs of file names

            .. code-block:: python
                :emphasize-lines: 1-6

                 input_files = [ [ 'A.1_start', 'A.2_start'],
                                 [ 'B.1_start', 'B.2_start'],
                                 [ 'C.1_start', 'C.2_start'],
                                 [ 'D.1_start', 'D.2_start'] ]

        The first job of:

            .. code-block:: python

                @combinations_with_replacement(input_files, formatter(), 3, ...)

        Will be

            .. <<python

            .. code-block:: python
                :emphasize-lines: 1-6

                # Two file pairs at a time
                ['A.1_start', 'A.2_start'],      # 0
                # versus itself
                ['A.1_start', 'A.2_start'],      # 1

            ..
                python

        First level of nesting:
            .. code-block:: python
                :emphasize-lines: 1-6

                ['A.1_start', 'A.2_start']  # [0]
                ['A.1_start', 'A.2_start']  # [1]

        Second level of nesting:
            .. code-block:: python
                :emphasize-lines: 1-6

                'A.2_start'                 # [0][1]
                'A.2_start'                 # [1][1]

        Parse filename without suffix
            .. code-block:: python
                :emphasize-lines: 1-6

                'A'                         # {basename[0][1]}
                'A'                         # {basename[1][1]}

        Python code:

            .. code-block:: python
                :emphasize-lines: 13,17,20,25,28-30

                from ruffus import *
                from ruffus.combinatorics import *

                #   initial file pairs
                @originate([ ['A.1_start', 'A.2_start'],
                             ['B.1_start', 'B.2_start'],
                             ['C.1_start', 'C.2_start'],
                             ['D.1_start', 'D.2_start']])
                def create_initial_files_ABCD(output_files):
                    for output_file in output_files:
                        with open(output_file, "w") as oo: pass

                #   @combinations_with_replacement
                @combinations_with_replacement(create_initial_files_ABCD,   # Input
                              formatter(),                                  # match input files

                              # tuple of 2 at a time
                              2,

                              # Output Replacement string
                              "{path[0][0]}/"
                              "{basename[0][1]}_vs_"
                              "{basename[1][1]}.combinations_with_replacement",

                              # Extra parameter: path for 1st set of files, 1st file name
                              "{path[0][0]}",

                              # Extra parameter. Basename for:
                              ["{basename[0][0]}",  # 1st set of files, 1st file name
                               "{basename[1][0]}",  # 2rd
                               ])
                def combinations_with_replacement_task(input_file, output_parameter,
                                                       shared_path, basenames):
                    print " - ".join(basenames)


                #
                #       Run
                #
                pipeline_run(verbose=0)


        This results in:

            .. code-block:: pycon

                >>> pipeline_run(verbose=0)
                A - A
                A - B
                A - C
                A - D
                B - B
                B - C
                B - D
                C - C
                C - D
                D - D


    **Parameters:**


.. _decorators.combinations_with_replacement.input:

    * **input** = *tasks_or_file_names*
       can be a:

       #.  Task / list of tasks.
            File names are taken from the |output|_ of the specified task(s)
       #.  (Nested) list of file name strings.
            File names containing ``*[]?`` will be expanded as a |glob|_.
             E.g.:``"a.*" => "a.1", "a.2"``

.. _decorators.combinations_with_replacement.filter:

.. _decorators.combinations_with_replacement.matching_formatter:

    * **filter** = *formater(...)*
       a :ref:`formatter<decorators.formatter>` indicator object containing optionally
       a  python `regular expression (re) <http://docs.python.org/library/re.html>`_.

.. _decorators.combinations_with_replacement.tuple_size:

    * **tuple_size** = *N*
        Select N elements at a time.

.. _decorators.combinations_with_replacement.output:

    * **output** = *output*
        Specifies the resulting output file name(s) after string substitution


.. _decorators.combinations_with_replacement.extras:

    * **extras** = *extras*
       Any extra parameters are passed verbatim to the task function

       If you are using named parameters, these can be passed as a list, i.e. ``extras= [...]``

       Any extra parameters are consumed by the task function and not forwarded further down the pipeline.

