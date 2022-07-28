
# Create a problem 

The first step is to choose the problem to be solved. 

In our [Marketplace Page](https://marketplace.qcentroid.xyz/) you can find information with all the problems available on our platform. For this tutorial we will assume that we want to solve the QRNG (Quantum Random Number Generation) problem (Our hello world quantum program!).

If you want to solve a new problem or modify an existing one, contact us here: info@qcentroid.xyz 

# Solver template

Once the problem has been choosen, let's see how we could create a solver associated to this problem in order to upload it to the Qapitan platform. First, we must prepare a Github repository with the following files:

- Solver documentation
- main.py
- app.py
- requirements.txt



### Solver documentation

You must include in the root of the repository a (name-solver).md file that will contain the documentation you want to include about your solver, explaining, if any, the parameters that can be inserted by the end user. Continuing with our example, we would have created MyFirstAlgorithm.md:
```txt
## MyFirstAlgorithm
Test documentation associated with my solver.
I don't have auxiliary parameters but I could define them like this:
- "parameter1: (int) This is what my first parameter does.
```


### main.py

This file will contain only a "run" function to which the parameters "input_data", "solver_params" and "extra_arguments" will be passed:
```python
from qiskit import QuantumCircuit, Aer, execute, IBMQ

def run(input_data, solver_params, extra_arguments):

    size = int(input_data['size'])
    backend = Aer.get_backend('qasm_simulator')

    qc = QuantumCircuit(1)
    qc.h(0)
    qc.measure_all()

    job = execute(qc, backend=backend, shots=size, memory=True)
    individual_shots = job.result().get_memory()

    output = ''
    for i in individual_shots:
        output+=i

    return output
```

### app.py

```python
import main
result = main.run(problem_data, solver_params, extra_arguments)
print(result)
```

This file purpose is for local testing only. In our platform it will be replaced with our own code that adds the necessary libraries and calls the right hardward providers.

*You should not modify this file to ensure your solver works in both local and production environments*

As you can see, from this file the only thing that will be done is to call the function "run" that we have just created in the main. 
In order to test that everything works correctly locally without having to upload it to the platform, this is the ideal place to do it. For this we must have clear how is the json that will be sent to your solver:

```json
{
    "data": {
        "size": 50
    },
    "solver_params":{
    },
    "extra_arguments": {
    }
}
```
with the corresponding "solver_params" and "extra_arguments" if any. 
Finally, we would only have to create an input.json like the file we have just shown and modify app.py as follows:

```python

input_file_name = "input.json"

# Input data loader. Container will get data from here

import json
with open(input_file_name) as f:
  dic = json.load(f)

# Optional extra parameters

if "extra_arguments" in dic:
    extra_arguments = dic['extra_arguments']
else:
    extra_arguments = {}

if "solver_params" in dic:
    solver_params = dic['solver_params']
else:
    solver_params = {}


import main
result = main.run(dic['input_data'], solver_params, extra_arguments)
print(result)

```

And when executing the app.py it will give us in this case a string of 50 zeros and ones created randomly (with the qiskit simulator).

### requeriments.txt

finally we must create the "requeriments.txt" indicating the libraries used as well as their versions:

```txt
qiskit==0.17.0
```

Any library from the standard approved ones in pip will be instaled. So make sure you add all your code dependencies.

It is very useful to create a new environment (with VirtualEnv or Conda) to make sure you don't have dependency mixes or you are not missing anything. Start with a completely new python3.8 environment and add all required modules to your requirements.txt file.

# Upload request
*Note: you will need to get access from your QCentroid rep first*
Once the repository has been defined with everything necessary, we must request the platform to upload it. To do this we must make a POST call to: https://api.qcentroid.xyz/solver/new (Check our API documentation page for more details) defined as follows:
```json
{
    "problem": "QRNG",
    "solver": "MyFirstAlgorithm",
    "repo_name":"MyRepo",
    "python_version":"3.7",
    "provider": ["local"],
    "solver_params": {},
    "extra_arguments":{}
}
```
- "problem": name of the problem we want to solve.

- "solver": will be the way in which the customer can refer to the algorithm.

- "provider": this variable is used to notify QCentroid that it will be necessary to use the credentials of a specific provider ("dwave", "ionq", "ibmq", ...). "local" means that no credentials will be needed.

- "solver_params": dictionary that will contain the variables you want to pass to your algorithm. These variables will be invisible to the end client.

- "extra_arguments": in certain situations, the customer may have the ability to modify certain parameters of the algorithm such as number of shots outside the problem definition itself. This dictionary will contain the set of such variables. The values assigned in this case will be taken as the default parameters in case the user decides not to enter them.

Once the request is made, we will check if you have access to the repository and if we cannot access it, we will send you an ssh key that you will have to insert in the repository configuration. Please note you need an active developer account in our platform for this process to work. If you don't have one contact us at info@qcentroid.xyz

Congratulations! with this little tutorial you have just seen how to create your first algorithm that you will be able to share on the QCentroid platform!
