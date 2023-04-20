# My Minimal Python Project Boilerplate

This is a minimal Python project boilerplate I use as a starting point for new Python projects.

## Prerequisites

- [pyenv](https://github.com/pyenv/pyenv/)
- [Poetry](https://python-poetry.org/)
- [pre-commit](https://pre-commit.com/)
- [tox](https://tox.readthedocs.io/)

I use [pipx](https://pypa.github.io/pipx/) to install Poetry, pre-commit, and tox globally.

## Installation

1. Clone this repository.
2. Add the following to your `.bashrc` or `.zshrc`. Note that you will need to change the value of `PYTHON_BOILERPLATE` to the path to the cloned repository.

    ```bash
    export PYTHON_BOILERPLATE="$HOME/sandbox/python-boilerplate/"

    # create a shell function `mkpythondir`
    mkpythondir() {( set -e
      local pyversion="${2:-$(pyenv version | cut -d' ' -f1)}"
      if [ ! -d "$(pwd)/$1" ]
      then
        rsync -r "$PYTHON_BOILERPLATE/" "$1" --exclude=".git" --exclude="README.md" # copy template files
        cd $1
        pyenv local $pyversion # set python version
        mkdir "${1//[-.]/_}" && touch "${1//[-.]/_}/__init__.py" # create package
        echo "# $1" > README.md # create README
        git init && pre-commit autoupdate && pre-commit install # git and pre-commit
        poetry init --name="$1" \
          --python="$pyversion" \
          --dev-dependency=flake8 \
          --dev-dependency=black \
          --dev-dependency=isort \
          --dev-dependency="docformatter[tomli]" \
          --dev-dependency=mypy \
          --no-interaction && \
          poetry env use $(pyenv version | cut -d' ' -f1) && \
          poetry install # poetry
        pyenv local --unset # unset python version
        git add . && git commit -m "Initial commit" # initial commit
      else
        echo "Directory $(pwd)/$1 already exists"
      fi
    )}
    ```

## Usage

Run `mkpythondir <project-name> [python-version]` to create a new project. If Python version is not specified, use the global Python version configured by pyenv or the local Python version (if configured by pyenv) in the current directory. The command does the following:

1. Create a directory with the project name and make it a git repository.
2. Create an empty readme.
3. Initialize pre-commit hooks for formatting ([black](https://black.readthedocs.io/), [isort](https://pycqa.github.io/isort/), and [docformatter](https://docformatter.readthedocs.io/)).
4. Configure tox to run linter ([flake8](https://flake8.pycqa.org/) and [mypy](https://mypy.readthedocs.io/)).
5. Create a Poetry virtual environment for the project with necessary development dependencies. I don't install black, isort, flake8, or mypy using pipx "globally" because different projects may requires different versions (you and your collaborators will hate each other if using different versions of black).
6. Make an initial commit.

## IDE Configuration

Related to the above but not necessary, I also use the following configuration in my IDE:

1. Run black and isort on save.
2. Enable mypy and flake8 linters.

This helps me to notice and fix issues before they are captured by pre-commit hooks or tox runs.
