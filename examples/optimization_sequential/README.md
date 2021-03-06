# Sequential optimization

This simple example illustrates how Phoenics can be used for the sequential optimization of a single objective function. We consider the Branin function for this illustration, which is non-convex with three global minima. 

### Dependencies
This example requires the following dependencies:
```python
tensorflow==1.4.1
edward==1.3.5
```
These dependencies can be installed with pip:
```bash
pip install tensorflow==1.4.1
pip install edward==1.3.5
```

## Configuration details

Configuration details for this optimization procedure are specified in `config.json`

```json
{
    "general": {
		"batches": 1,
		"parallel": true,
		"boosted":  true,
		"backend": "tfprob",
		"sampling_strategies": 2
    },
    "parameters": [
        {"name": "x", "type": "continuous", "low": -5.0, "high": 10.0, "size": 1},
        {"name": "y", "type": "continuous", "low":  0.0, "high": 15.0, "size": 1}
    ],
    "objectives": [
        {"name": "branin", "goal": "minimize"}
    ]
}

```

### "general" 
This section contains general information about the optimization procedure. You can choose the probabilistic modeling library, number and size of parameter batches returned from one query, as well as parallelization options to accerate optimization runs.  

| Parameter              | Description                         											       | 
|:-----------------------|:------------------------------------------------------------------------------------|
| `backend`              | `string`: BNN backend type (more info [here](https://github.com/ChemOS-Inc/phoenics/blob/master/README.md#dependencies-and-requirements))  										       |
| `batches`             | `int`: Number of batches returned 										       | 
| `sampling_strategies`              | `int`: Number of parameter sets per batch  											       | 
| `parallel`             | `bool`: Parallel execution of the computations 										       | 
| `boosted`              | `bool`: Approximation to the mathematical model to accelerate the calculations								       | 

**Note**: The parameter sets in one batch are proposed based on different sampling strategies favoring exporative or exploitative behavior to various degrees. For details, see [examples/optimization_parallel](https://github.com/aspuru-guzik-group/phoenics/tree/master/examples/optimization_parallel).


### "parameters" 

This section contains information about the parameter space to be scanned. Each new parameter is defined via its name (`"x"` and `"y"`) and has to be given a lower bound (`"low"`) and an upper bound (`"high"`). The `"size"` parameter defines the number of elements in the parameter vector. Note, that the only currently supported parameter `"type"` is `"continuous"`. 

**Note**: Defining a single parameter `"param"` of with a `"size"` of 12 is equivalent to defining 12 separate parameters `"param_0"` to `"param_11"` of `"size"` one with identical lower and upper bounds.

### "objectives"

This section contains information about the objectives. Each objective is identified via its name (`"branin"`). For a single-objective optimization run, set `"hierarchy"` to zero and `"tolerances"` to zero. You can choose the objective `"type"` from `"minimize"` and `"maximize"`. For more information see the multi-objective optimization example ([examples/optimization_multiple_objectives](https://github.com/aspuru-guzik-group/phoenics/tree/master/examples/optimization_multiple_objectives))


## Running the optimization procedure

To start a new optimization procedure, we first need to initialize a Phoenics instance from a configuration file 

```python
from phoenics import Phoenics
phoenics = Phoenics('config.json')
```

We can then create a loop in which we query our Phoenics instance for new parameters, evaluate the objective value for the proposed parameters, and store the observations for future queries

```python
observations = []
for num_iter in range(max_iter):
	
    # query for new parameters
    params = phoenics.recommend(observations = observations)
    
    # evaluate the proposed parameters
    for param in params:
     	observation = merit_function(param)
        observations.append(observation)
```

This general procedure is outlined in `optimize_branin.py`, where we additionally log the proposed parameters and associated objective values at each iteration of the optimization procedure. 

**Note**: The merit function can be substituted with any arbitrary mechanism to assess the merit of the proposed parameter sets, such as computations or experimentation (see [examples/application_robot_calibration](https://github.com/aspuru-guzik-group/phoenics/tree/master/examples/application_robot_calibration)). The merit-function should be defined such that it stores objective values in the parameter dictionary where the keys are set to be the objective names defined in the configuration file. An example is provided in `branin.py`.
