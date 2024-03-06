To make the [eWatercycle infrastructure](https://github.com/eWaterCycle/infra) usable for teaching we need to add stuff to it or change it.

Below is a list of possible stuff for teaching:

# Books & instances

* https://jupyter4edu.github.io/jupyter-edu-book/
* https://www.surf.nl/en/services/jupyter-for-education

# nbgrader 

https://nbgrader.readthedocs.io/

nbgrader with R:
- https://github.com/ttimbers/jupyter-nbgrader-r
- https://doc.cocalc.com/howto/nbgrader.html#nbgrader-for-r

# otter-grader

https:/otter-grader.readthedocs.io/

# OK

Server: Hosted Web Application to store submissions
Client: Enables students to run tests locally and submit
Autograding: Secure grading of student code.

https://okpy.org/

# e2x

* https://github.com/DigiKlausur/e2xhub - A JupyterHub extension for simplifying course management for teaching and exams
* https://github.com/DigiKlausur/e2xgrader - e2xgrader is an add-on for nbgrader that adds functionality for teachers and students. e2xgrader introduces new cell types and tools for graders (per question grading view, authoring component, pen-based grading) and students (assignment toolbar, exam toolbar, restricted notebook extension).


# Brightspace

* https://docs.valence.desire2learn.com/reference.html - API docs
* https://github.com/Brightspace/valence-sdk-python - Python Client
* https://www.imsglobal.org/spec/lti/v1p3 - Learning Tools Interoperability
* https://community.d2l.com/brightspace/kb/articles/1134-brightspace-api-authentication-guide-oauth-2-0 - For authentication & authorization

# Commercial

## illumidesk

https://github.com/IllumiDesk/illumidesk

## Codegrade

https://www.codegrade.com/

Looks good (nice lms integration), however seems to be more focussed on students running code on their own machines. The testing environment is apparently fully configurable, but there doesn't seem to be a clear way to have users work in a notebook that is hosted remotely.

## Vocareum

https://www.vocareum.com/

Already available at TU Delft, but apparently not enough compute available https://www.tudelft.nl/teaching-support/educational-tools/vocareum

# cocalc

CoCalc is a virtual online workspace for calculations, research, collaboration and authoring documents. Your web browser is all you need to escape the confined space of your desktop and move to the cloud. This guide explains the features of CoCalc in depth and shows how you can use them productively.

https://cocalc.com/

Their code is open source, but does not have clear setup instructions. Best to [pay them](https://cocalc.com/pricing/onprem) for running it ourselves.

# Nbgrader extensions

https://github.com/LibreTexts/ngshare - nbgrader sharing service

# Jupyter extensions

* https://github.com/jupyterhub/hubshare - A directory sharing service for JupyterHub
* https://github.com/jupyterhub/zero-to-jupyterhub-k8s/issues/174 - nbgrader support in JupyterHub on Kubernetes

