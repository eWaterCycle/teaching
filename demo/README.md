# Testing out nbgrader quickstart

```shell
python3 -m venv venv
pip install -r requirements.txt
mkdir exchange
```

```shell
nbgrader quickstart hydro-mod-1
cd hydro-mod-1
```

To `nbgrader_config.py` add `c.Exchange.root = "<cwd>/../exchange"
`

```
jupyter lab
```

## Create student version

```shell
mkdir -p submitted/student1
cp -r release/* submitted/student1/
```

## JupyterHub

Follow https://nbgrader.readthedocs.io/en/stable/configuration/jupyterhub_config.html#example-use-case-multiple-classes
and
https://github.com/jupyter/nbgrader/tree/main/demos/demo_multiple_classes

```
docker build -t nbgraderhub .
docker run -p 8000:8000 nbgraderhub
```

Goto http://localhost:8000

Login with
* instructor1: instructor1
* student1: student1

To edit assignemt
1. As instructor1
1. Goto Nbgrader -> Courses -> select course101
1. Authorize yourself
1. You can now edit the ps1 assigment in Formgrader

To fetch assigment

1. As student1
1. Goto Nbgrader -> Assignment list
1. Fetch ps1

# TODO

* [x] JupyterHub
* [x] Multiple users
* [x] Multiple courses
* R
* oauth or user sync
* 