
# Learning Pixi

[Proposal Design](https://pixi.sh/latest/design_proposals/multi_environment_proposal/)

- environment set mechanism 
- clear, conflict free management of dependencies tailored to specific environments while also maintaining the integrity of fixed lockfiles

There are multiple scenarios where multiple environments are useful.

- Testing of multiple package versions, e.g. py39 and py310 or polars 0.12 and 0.13.
- Smaller single tool environments, e.g. lint or docs.
- Large developer environments, that combine all the smaller environments, e.g. dev.
- Strict supersets of environments, e.g. prod and test-prod where test-prod is a strict superset of prod.
- Multiple machines from one project, e.g. a cuda environment and a cpu environment.

## Features and Environments

`dependencies`: The conda package dependencies
`pypi-dependencies`: The pypi package dependencies
`system-requirements`: The system requirements of the environment
`activation`: The activation information for the environment
`platforms`: The platforms the environment can be run on.
`channels`: The channels used to create the environment. Adding the priority field to the channels to allow concatenation of channels instead of overwriting.
`target`: All the above features but also separated by targets.
`tasks`: Feature specific tasks, tasks in one environment are selected as default tasks for the environment.

``` toml
[dependencies] # short for [feature.default.dependencies]
python = "*"
numpy = "==2.3"

[pypi-dependencies] # short for [feature.default.pypi-dependencies]
pandas = "*"

[system-requirements] # short for [feature.default.system-requirements]
libc = "2.33"

[activation] # short for [feature.default.activation]
scripts = ["activate.sh"]
```

``` toml
[feature.py39.dependencies]
python = "~=3.9.0"
[feature.py310.dependencies]
python = "~=3.10.0"
[feature.test.dependencies]
pytest = "*"
```

Define tasks as defaults of an environment
``` toml
[feature.test.tasks]
test = "pytest"

[environments]
test = ["test"]
# `pixi run test` == `pixi run --environment test test`
```

Testing a production envirronment with additional dependencies
``` toml
[environments]
# Creating a `prod` environment which is the minimal set of dependencies used for production.
prod = {features = ["py39"], solve-group = "prod"}
# Creating a `test_prod` environment which is the `prod` environment plus the `test` feature.
test_prod = {features = ["py39", "test"], solve-group = "prod"}
# Using the `solve-group` to solve the `prod` and `test_prod` environments together
# Which makes sure the tested environment has the same version of the dependencies as the production environment.
```


## User Interface Environment Activation

Users can manually activate the desired environment via command line or configuration

- guarantees a conflict-free environment by allowing only one feature set to be active at a time
- for the user, the cli would look like this:

*default*
``` shell
pixi run python
# Runs python in the 'default' environment
```

*specific environment*
``` shell
pixi run -e test pytest
pixi run --environment test pytest
# Runs pytest in the 'test' environment
```

*activating a shell in an environment*
``` shell
pixi shell -e cuda
pixi shell --environment cuda
```

*running any command in an environment*
``` shell
pixi run -e env python
pixi run --environment env python
```

## 7 reasons to switch from conda to pixi

[link](https://prefix.dev/blog/pixi_a_fast_conda_alternative)

First of all its **Faster** - Pixi is 10x faster than conda

- integrated with pypi more deeply, which allows for faster dependency resolution

Painlessly switch from conda to pixi

- A single command `pixi init --import ./myenv.yml` should bring all dependencies from an existing conda `environment.yml` file to a newly created `pixi.toml`.

### Reason 1: Work Faster
![alt text](image.png)

### Reason 2: Better Integration with PyPi

- deeper resolution using Rust's `uv` resolver 
- pypi packages are also locked alongside conda


### Reason 3: No more miniconda base environment

pixi is a statically linked binary, which means it doesn't need a base environment to run.

- optimized for speed and compatability 
- no messy base environment or miniconda/miniforge shell script installation

``` shell
# Linux & macOS
curl -fsSL https://pixi.sh/install.sh | bash
# Windows
iwr -useb https://pixi.sh/install.ps1 | iex
```


### Reason 4: Install your favorite tools globally

conda-forge tools such as git, bat, rg, bash can be installed globally with pixi

each globally installed tool lives in its own isolated environment 


``` shell
pixi global install git bash ripgrep bat
```

### Reason 5: Sidestep activation

no "conda activate" or "conda deactivate" needed
- project should define all configurations in a `pixi.toml` file and run pixi shell or pixi run 


### Reason 6: Use tasks to easily collaborate

unique to Pixi are `tasks` which are simple entry points to a project such as `start` `build` `lint`
Instead of writing `Makefiles` or bash scripts with complicated installation instructions you can make
everything work cross-platform with pixi tasks

### Reason 7: Native lockfiles

Pixi natively creates and uses lockfiles with lots of inspiration from conda-lock, poetry, cargo, and other 
modern package managers.













