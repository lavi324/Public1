Open a new GitHub repo, pull the repo into the cloud shell and config the GitHub user for a future Push requests.
Enable "Kubernetes Engine API" (for GCP).
Create the "production" namespace that gonna be used (the rest of the namespaces will be created in the future Terraform script).
Add the Helm repo's that gonna be used in the future Terraform script and make sure these are the latest chart versions.
Set up a git ignore file that will prevent a high memory Terraform files to be pushed. 
Plan and Apply Terraform script.
Access Your API Key in https://site.financialmodelingprep.com/developer/docs/dashboard
Create two directories called "backend" and "frontend".
Install Express (backend web application framework for building RESTful APIs with Node.js) in the backend directory and ensure that all the default folder Structure were created.
Install axios (HTTP Client for node.js and the browser), cors(security mechanism implemented by web browsers to prevent unauthorized access to resources on a web page from different origins) and mongoose (Object Data Modeling library for MongoDB).
Set up the backend file and the Docker file.
Install React (JavaScript library for building user interfaces) in the frontendend directory and ensure that all the default folder Structure were created.
Install axios again.
Set up the frontend file and the Docker file.
Create 3 Docker hub repositories (frontend, backend and frontend Helm chart).
Build and Push the Docker images.
Create Kubernetes files with the new Docker images and apply.
