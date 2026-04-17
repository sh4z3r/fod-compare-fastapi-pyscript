# fod-compare-fastapi-pyscript
Pycon Challenge 2023


## Tech Stack
Client: PyScript, HTML, JS, CSS.
Server: FastAPI on AWS

This PoC was created to compare two scans from Fortify on Demand (FoD). It will retrieve the results and generate tasks for the files with changes to address.

Just so you know, this demo uses sample data. If you intend to work with actual scans, you will need to provide your own FoD keys in the **.config.json** file.

The JSON file on the Backend can be generated with FoD-Compare.
https://github.com/youngcs97/FoD-Compare

## Demo
Demo is hosted in AWS. 

## Installation and running

### Server:
Install the dependencies and run FASTAPI

```console
pip install -r requirements.txt
uvicorn fod-sample-info:app --host 0.0.0.0 --port 8000 --reload
```
Another option is to use the Dockerfile:
```console
 docker build . --tag api.fod:2023
 docker run -d -p 8000:8000 api.fod:2023
```
### Client:
The frontend is HTML, so any webserver would work. You can use Python builtin web server.

```console
cd web
python -m http.server -b 0.0.0.0 80
```
Or use the Dockerfile:
```console
 docker build -t fronted-fod .
 docker run -d -p 80:80 fronted-fod
 ```
 
## Roadmap / To-Do
[ ] Enhance the UI/UX for better task visualization.

[ ] Add export functionality for generated tasks (PDF/CSV).

## Credits
- FastAPI - https://fastapi.tiangolo.com/lo/
- Pyscript Example - https://pyscript.net/examples/todo.html
- FoD Compare by Chris Young - https://github.com/youngcs97/FoD-Compare
- Challenge_pycon2023 Demo - https://bitbucket.org/endava-pycon2023/challenge_pycon2023/src/master/
