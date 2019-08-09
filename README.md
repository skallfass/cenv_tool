# conda-env-manager: cenv

![coverage](docs/img/coverage.svg)
[![PyPI version fury.io](https://badge.fury.io/py/ansicolortags.svg)](https://pypi.python.org/pypi/cenv_tool/)
[![PyPI pyversions](https://img.shields.io/pypi/pyversions/ansicolortags.svg)](https://pypi.python.org/pypi/cenv_tool/)
[![MIT license](https://img.shields.io/badge/License-MIT-blue.svg)](https://lbesson.mit-license.org/)

![logo](docs/img/logo.png)

Due to the redundant dependency information inside the `meta.yaml` (required
to create the conda-package) and the `environment.yml` (as definition file
for the conda-environment during development and for production), `cenv`
(short form for `conda-env-manager`) was created to make the `meta.yaml`
the only relevant file and to create and update conda-environment from the
definition inside this `meta.yaml`.
The name of the conda-environment to create / update is defined in the section
`extra` and the variable `env_name` inside the `meta.yaml`.

The steps run by cenv:

* creation of a backup if the environment already exists followed by the
  removal of the previous environment.
* creation of the environment as defined in the `meta.yaml`.
  If any failures occurred during creation and the backup was created, the
  command to reset the backup-version can be used.
* if enabled in the config file the environment.yml is exported after creation
  / update of the environment.


The usage of cenv reduces the conda commands to use to the following:

* `conda activate ...` to activate the environment
* `conda deactivate` to deactivate an environment
* `conda info` to show information about the currently activated environment
* `conda search ...` to search for availability of a package in the conda
  channels.
* `conda remove -n ... --all` to remove an environment
* `cenv` to create / update an environment


## Installation

Install `cenv` using pip:
```bash
pip3 install cenv_tool
```

Now run `init_cenv` to create the relevant config-files and add the
autoactivate- and autoupdate-shell-function to your `.bashrc` / `.zshrc`.


### autoactivate and autoupdate

Per default these features are deactivated, even if added to your shell by
running `init_cenv`.


#### autoactivate-feature

The autoactivate-feature activates the conda-environment as named
`extra`-section in the meta.yaml located at `conda-build/meta.yaml`, if the
environment exists.
To activate the autoactivate-features run:
```bash
autoactivate_toggle
```

#### autoupdate-feature

The autoupdate checks if the content of the meta.yaml changed.
The current state is stored as a md5sum in `conda-build/meta.md5`.
If it changed the cenv-process is called.

For the autoupdate-feature run:
```bash
autoupdate_toggle
```


## Usage

All steps required to create or update the projects conda environment are
run automatically running:
```bash
cenv
```

**ATTENTION**:
>    If you use cenv, each environment should only be created, updated and
>    modified using `cenv`!
>    This means the commands `conda install`, `conda remove` are not used
>    anymore.
>    Changes of the dependencies of the environment are defined inside the
>    `meta.yaml` and are applied by using `cenv`.
>
>    This means:
>
>    * new dependency required => add it in `meta.yaml` and run `cenv`.
>    * dependency not needed anymore => remove it from `meta.yaml` and run
>      `cenv`.
>    * need of another version of dependency => change the version of dependency
>      in `meta.yaml` and run `cenv`.

The required information about the projects conda environment are extracted
from the meta.yaml.
This meta.yaml should be located inside the project folder at
`./conda-build/meta.yaml`.
The project-configuration is defined in the `extra` section of the `meta.yaml`.
There you can define the name of the projects conda-environment at
`env_name`.
Also you can define requirements only needed during development but not to be
included into the resulting conda package.
These requirements have to be defined in the `dev_requirements`-section.

All other parts of the `meta.yaml` have to be defined as default.

A meta.yaml valid for cenv should look like the following:
```yaml
    {% set data = load_setup_py_data() %}

    package:
        name: "example_package"
        version: {{ data.get("version") }}

    source:
        path: ..

    build:
        build: {{ environ.get('GIT_DESCRIBE_NUMBER', 0) }}
        preserve_egg_dir: True
        script: python -m pip install --no-deps --ignore-installed .

    requirements:
        build:
          - python 3.6.8
          - pip
          - setuptools
        run:
          - python 3.6.8
          - attrs >=18.2
          - jinja2 >=2.10
          - ruamel.yaml >=0.15.23
          - six >=1.12.0
          - yaml >=0.1.7
          - marshmallow >=3.0.0rc1*

    test:
        imports:
            - example_package

    extra:
        env_name: example
        dev_requirements:
            - ipython >=7.2.0
```

**ATTENTION**:
>    In the `requirements-run-section` the minimal version of each package
>    has to be defined!
>    The same is required for the `dev_requirements`-section.
>    Not defining a version will not create or update a conda-environment,
>    because this is not the purpose of the conda-usage.
>    The validity of the `meta.yaml` is checked in `cenv` using the
>    `marshmallow` package.
>    You can additionally add upper limits for the version like the following:
>    `- package >=0.1,<0.3`

If cenv is run the environment is created / updated from the definition inside
this `meta.yaml`.
The creation of the backup of the previous environment ensures to undo changes
if any error occurs during recreation of the environment.


**ATTENTION**:
>    `cenv` can only update the environment if it is not activated.
>    So ensure the environment to be deactivated before running `cenv`.

Per default exporting the conda environment definition into an environment.yml
is turned off.
If you want to turn this functionality on you need to modify your
`~/.config/cenv.yml` as described in the configuration-part.

Example for the output of the `cenv` command:

```bash
    ┣━━ Cloning existing env as backup ...
    ┣━━ Removing existing env ...
    ┣━━ Creating env ...
    ┣━━ Removing backup ...
    ┗━━ Exporting env to environment.yml ...
```

# Development of cenv

To create the environment to develop cenv run the pre-commit hooks manually:
```bash
pyenv local 3.7.3
pre-commit run --all-files
poetry shell
```
