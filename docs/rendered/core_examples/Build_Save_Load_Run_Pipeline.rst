Build, Save, Load, and Run a Pipeline
=====================================

**Author:** Eric Charles

**Last Run Successfully:** December 22, 2023

This notebook shows how to:

1. Build a simple interactive rail pipeline,

2. Save that pipeline (including configuration) to a yaml file,

3. Load that pipeline from the saved yaml file,

4. Run the loaded pipeline.

.. code:: ipython3

    import os
    import numpy as np
    import ceci
    import rail
    from rail.core.stage import RailStage
    from rail.creation.degraders.spectroscopic_degraders import LineConfusion
    from rail.creation.degraders.quantityCut import QuantityCut
    from rail.creation.degraders.photometric_errors import LSSTErrorModel
    from rail.creation.engines.flowEngine import FlowCreator, FlowPosterior
    from rail.core.data import TableHandle
    from rail.core.stage import RailStage
    from rail.tools.table_tools import ColumnMapper, TableConverter

We’ll start by setting up the RAIL data store. RAIL uses
`ceci <https://github.com/LSSTDESC/ceci>`__, which is designed for
pipelines rather than interactive notebooks; the data store will work
around that and enable us to use data interactively.

When working interactively, we want to allow overwriting data in the
RAIL data store to avoid errors if we re-run cells.

See the ``rail/examples/goldenspike_examples/goldenspike.ipynb`` example
notebook for more details on the Data Store.

.. code:: ipython3

    DS = RailStage.data_store
    DS.__class__.allow_overwrite = True

Build the pipeline
------------------

Some configuration setup
~~~~~~~~~~~~~~~~~~~~~~~~

The example pipeline builds some of the RAIL creation functionality into
a pipeline.

Here we are defining:

1. The location of the pretrained PZFlow file used with this example.

2. The bands we will be generating data for.

3. The names of the columns where we will be writing the error
   estimates.

4. The grid of redshifts we use for posterior estimation.

.. code:: ipython3

    from rail.utils.path_utils import find_rail_file
    
    flow_file = find_rail_file("examples_data/goldenspike_data/data/pretrained_flow.pkl")
    bands = ["u", "g", "r", "i", "z", "y"]
    band_dict = {band: f"mag_{band}_lsst" for band in bands}
    rename_dict = {f"mag_{band}_lsst_err": f"mag_err_{band}_lsst" for band in bands}
    post_grid = [float(x) for x in np.linspace(0.0, 5, 21)]

Define the pipeline stages
~~~~~~~~~~~~~~~~~~~~~~~~~~

The RailStage base class defines the ``make_stage`` “classmethod”
function, which allows us to make a stage of that particular type in a
general way.

Note that that we are passing in the configuration parameters to each
pipeline stage as keyword arguments.

The names of the parameters will depend on the stage type.

A couple of things are important:

1. Each stage should have a unique name. In ``ceci``, stage names
   default to the name of the class (e.g., FlowCreator, or
   LSSTErrorModel); this would be problematic if you wanted two stages
   of the same type in a given pipeline, so be sure to assign each stage
   its own name.

2. At this point, we aren’t actually worrying about the connections
   between the stages.

.. code:: ipython3

    flow_engine_test = FlowCreator.make_stage(
        name="flow_engine_test", model=flow_file, n_samples=50
    )
    
    lsst_error_model_test = LSSTErrorModel.make_stage(
        name="lsst_error_model_test", renameDict=band_dict
    )
    
    col_remapper_test = ColumnMapper.make_stage(
        name="col_remapper_test", hdf5_groupname="", columns=rename_dict
    )
    
    flow_post_test = FlowPosterior.make_stage(
        name="flow_post_test", column="redshift", flow=flow_file, grid=post_grid
    )
    
    table_conv_test = TableConverter.make_stage(
        name="table_conv_test", output_format="numpyDict", seed=12345
    )


.. parsed-literal::

    Inserting handle into data store.  model: /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/rail/examples_data/goldenspike_data/data/pretrained_flow.pkl, flow_engine_test


.. code:: ipython3

    flow_engine_test.sample(6, seed=0).data


.. parsed-literal::

    Inserting handle into data store.  output_flow_engine_test: inprogress_output_flow_engine_test.pq, flow_engine_test




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>mag_z_lsst</th>
          <th>mag_y_lsst</th>
          <th>mag_r_lsst</th>
          <th>mag_i_lsst</th>
          <th>redshift</th>
          <th>mag_u_lsst</th>
          <th>mag_g_lsst</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>22.734173</td>
          <td>22.580509</td>
          <td>24.027840</td>
          <td>22.996883</td>
          <td>0.739296</td>
          <td>25.822563</td>
          <td>25.088484</td>
        </tr>
        <tr>
          <th>1</th>
          <td>22.666660</td>
          <td>22.415169</td>
          <td>23.995960</td>
          <td>23.025354</td>
          <td>0.644894</td>
          <td>27.391764</td>
          <td>25.447910</td>
        </tr>
        <tr>
          <th>2</th>
          <td>21.013422</td>
          <td>20.779903</td>
          <td>21.757725</td>
          <td>21.298742</td>
          <td>0.348834</td>
          <td>23.999668</td>
          <td>22.884043</td>
        </tr>
        <tr>
          <th>3</th>
          <td>24.050123</td>
          <td>23.818220</td>
          <td>24.271830</td>
          <td>24.152014</td>
          <td>1.623727</td>
          <td>24.338676</td>
          <td>24.272142</td>
        </tr>
        <tr>
          <th>4</th>
          <td>23.140982</td>
          <td>23.095510</td>
          <td>23.577923</td>
          <td>23.190630</td>
          <td>0.551647</td>
          <td>25.136570</td>
          <td>24.586559</td>
        </tr>
        <tr>
          <th>5</th>
          <td>21.022926</td>
          <td>20.927565</td>
          <td>22.416426</td>
          <td>21.435768</td>
          <td>0.839687</td>
          <td>23.225256</td>
          <td>23.068544</td>
        </tr>
      </tbody>
    </table>
    </div>



Make the pipeline and add the stages
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Here we make an empty interactive pipeline (interactive in the sense
that it will be run locally, rather than using the batch submission
mechanisms built into ``ceci``), and add the stages to that pipeline.

.. code:: ipython3

    pipe = ceci.Pipeline.interactive()
    stages = [flow_engine_test, lsst_error_model_test, col_remapper_test, table_conv_test]
    for stage in stages:
        pipe.add_stage(stage)

Interactive introspection
~~~~~~~~~~~~~~~~~~~~~~~~~

Here are some examples of interactive introspection into the pipeline

I.e., some functions that you can use to figure out what the pipeline is
doing.

.. code:: ipython3

    # Get the names of the stages
    pipe.stage_names




.. parsed-literal::

    ['flow_engine_test',
     'lsst_error_model_test',
     'col_remapper_test',
     'table_conv_test']



.. code:: ipython3

    # Get the configuration of a particular stage
    pipe.flow_engine_test.config




.. parsed-literal::

    StageConfig{output_mode:default,n_samples:6,seed:0,name:flow_engine_test,model:/opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/rail/examples_data/goldenspike_data/data/pretrained_flow.pkl,config:None,}



.. code:: ipython3

    # Get the list of outputs 'tags'
    # These are how the stage thinks of the outputs, as a list names associated to DataHandle types.
    pipe.flow_engine_test.outputs




.. parsed-literal::

    [('output', rail.core.data.PqHandle)]



.. code:: ipython3

    # Get the list of outputs 'aliased tags'
    # These are how the pipeline things of the outputs, as a unique key that points to a particular file
    pipe.flow_engine_test._outputs




.. parsed-literal::

    {'output_flow_engine_test': 'output_flow_engine_test.pq'}



Connect up the pipeline stages
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We can use the ``RailStage.connect_input`` function to connect one stage
to another. By default, this will connect the output data product called
``output`` for one stage.

.. code:: ipython3

    lsst_error_model_test.connect_input(flow_engine_test)
    col_remapper_test.connect_input(lsst_error_model_test)
    # flow_post_test.connect_input(col_remapper_test, inputTag='input')
    table_conv_test.connect_input(col_remapper_test)


.. parsed-literal::

    Inserting handle into data store.  output_lsst_error_model_test: inprogress_output_lsst_error_model_test.pq, lsst_error_model_test
    Inserting handle into data store.  output_col_remapper_test: inprogress_output_col_remapper_test.pq, col_remapper_test


Initialize the pipeline
~~~~~~~~~~~~~~~~~~~~~~~

This will do a few things:

1. Attach any global pipeline inputs that were not specified in the
   connections above. In our case, the input flow file is pre-existing
   and must be specified as a global input.

2. Specifiy output and logging directories.

3. Optionally, create the pipeline in ‘resume’ mode, where it will
   ignore stages if all of their output already exists.

.. code:: ipython3

    pipe.initialize(
        dict(model=flow_file), dict(output_dir=".", log_dir=".", resume=False), None
    )




.. parsed-literal::

    (({'flow_engine_test': <Job flow_engine_test>,
       'lsst_error_model_test': <Job lsst_error_model_test>,
       'col_remapper_test': <Job col_remapper_test>,
       'table_conv_test': <Job table_conv_test>},
      [<rail.creation.engines.flowEngine.FlowCreator at 0x7fab6f8764d0>,
       <rail.creation.degraders.photometric_errors.LSSTErrorModel at 0x7fabd483ab60>,
       Stage that applies remaps the following column names in a pandas DataFrame:
       f{str(self.config.columns)},
       <rail.tools.table_tools.TableConverter at 0x7fabd483b3d0>]),
     {'output_dir': '.', 'log_dir': '.', 'resume': False})



Save the pipeline
-----------------

This will actually write two files (as this is what ``ceci`` wants)

1. ``pipe_example.yml``, which will have a list of stages, with
   instructions on how to execute the stages (e.g., run this stage in
   parallel on 20 processors). For an interactive pipeline, those
   instructions will be trivial.

2. ``pipe_example_config.yml``, which will have a dictionary of
   configurations for each stage.

.. code:: ipython3

    pipe.save("pipe_saved.yml")

Read the saved pipeline
-----------------------

.. code:: ipython3

    pr = ceci.Pipeline.read("pipe_saved.yml")

Run the newly read pipeline
---------------------------

This will actually launch a Unix process to individually run each stage
of the pipeline; you can see the commands that are being executed in
each case.

.. code:: ipython3

    pr.run()


.. parsed-literal::

    
    Executing flow_engine_test
    Command is:
    OMP_NUM_THREADS=1   python3 -m ceci rail.creation.engines.flowEngine.FlowCreator   --model=/opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/rail/examples_data/goldenspike_data/data/pretrained_flow.pkl   --name=flow_engine_test   --config=pipe_saved_config.yml   --output=./output_flow_engine_test.pq 
    Output writing to ./flow_engine_test.out
    


.. parsed-literal::

    Job flow_engine_test has completed successfully!


.. parsed-literal::

    
    Executing lsst_error_model_test
    Command is:
    OMP_NUM_THREADS=1   python3 -m ceci rail.creation.degraders.photometric_errors.LSSTErrorModel   --input=./output_flow_engine_test.pq   --name=lsst_error_model_test   --config=pipe_saved_config.yml   --output=./output_lsst_error_model_test.pq 
    Output writing to ./lsst_error_model_test.out
    


.. parsed-literal::

    Job lsst_error_model_test has failed with status 1


.. parsed-literal::

    
    *************************************************
    Error running pipeline stage lsst_error_model_test.
    
    Standard output and error streams in ./lsst_error_model_test.out
    *************************************************




.. parsed-literal::

    1



Running saved pipelines from the command line
---------------------------------------------

Once you’ve saved a pipeline and have the ``pipeline_name.yml`` and
``pipeline_name_config.yml`` file pair, you can go ahead and run the
pipeline from the command line instead, if you prefer. With
`ceci <https://github.com/LSSTDESC/ceci>`__ installed in your
environment, just run ``ceci path/to/the/pipeline.yml``. Running the
pipeline we’ve just made would look like:

.. code:: ipython3

    ! ceci pipe_saved.yml


.. parsed-literal::

    /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/pty.py:89: RuntimeWarning: os.fork() was called. os.fork() is incompatible with multithreaded code, and JAX is multithreaded, so this will likely lead to a deadlock.
      pid, fd = os.forkpty()


.. parsed-literal::

    Inserting handle into data store.  model: /opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/rail/examples_data/goldenspike_data/data/pretrained_flow.pkl, flow_engine_test


.. parsed-literal::

    
    Executing flow_engine_test
    Command is:
    OMP_NUM_THREADS=1   python3 -m ceci rail.creation.engines.flowEngine.FlowCreator   --model=/opt/hostedtoolcache/Python/3.10.16/x64/lib/python3.10/site-packages/rail/examples_data/goldenspike_data/data/pretrained_flow.pkl   --name=flow_engine_test   --config=pipe_saved_config.yml   --output=./output_flow_engine_test.pq 
    Output writing to ./flow_engine_test.out
    


.. parsed-literal::

    Job flow_engine_test has completed successfully!
    
    Executing lsst_error_model_test
    Command is:
    OMP_NUM_THREADS=1   python3 -m ceci rail.creation.degraders.photometric_errors.LSSTErrorModel   --input=./output_flow_engine_test.pq   --name=lsst_error_model_test   --config=pipe_saved_config.yml   --output=./output_lsst_error_model_test.pq 
    Output writing to ./lsst_error_model_test.out
    


.. parsed-literal::

    Job lsst_error_model_test has failed with status 1
    
    *************************************************
    Error running pipeline stage lsst_error_model_test.
    
    Standard output and error streams in ./lsst_error_model_test.out
    *************************************************
    Pipeline failed.  No joy sparked.

