
# Create a problem 

The first step is to choose the problem to be solved. Here (/// enlace a postman ///) you can find information with all the problems available on our platform. For this tutorial we will assume that we want to solve the QRNG (Quantum Random Number Generation) problem.

# Solver template

Once the problem has been choosen, let's see how we could create a solver associated to this problem in order to upload it to the Qapitan platform. First, we must prepare a Github repository with the following files:

- solver documentation
- main.py
- app.py
- requirements.txt


### solver documentation

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
As you can see, from this file the only thing that will be done is to call the function "run" that we have just created in the main. However, if you want to test that everything works correctly without having to upload it to the platform, this is the ideal place to do it. For this we must have clear how is the json that will be sent to your solver:

```json
{
    "input_data": {
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
#########  THIS FILE WILL BE REPLACED  #########

input_file_name = "input.json"

################################################
#########    DO NOT TOUCH FROM HERE    #########
################################################

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

# Upload request
Once the repository has been defined with everything necessary, we must request the platform to upload it. To do this we must make the following POST (/// link to postman////) defined as follows:
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

- "provider": this variable is used to notify Qapitan that it will be necessary to use the credentials of a specific provider ("dwave", "ionq", "ibmq", ...). "local" means that no credentials will be needed.

- "solver_params": dictionary that will contain the variables you want to pass to your algorithm. These variables will be invisible to the end client.

- "extra_arguments": in certain situations, the customer may have the ability to modify certain parameters of the algorithm such as number of shots outside the problem definition itself. This dictionary will contain the set of such variables. The values assigned in this case will be taken as the default parameters in case the user decides not to enter them.

Once the request is made, we will check if you have access to the repository and if we cannot access it, we will send you an ssh key that you will have to insert in the repository configuration.

Congratulations! with this little tutorial you have just seen how to create your first algorithm that you will be able to share on the Qapitan platform!
