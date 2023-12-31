# Become a Head Chef

This small Data Engineering project focuses on collecting, processing, and presenting recipe data from the Allrecipes website. The goal is to build a structured dataset containing recipe titles, nutrition facts, photos, prep and cook times, and more. The project will involve web scraping, data extraction, and data transformation, preparing the groundwork for a user-friendly web app.

Help from:

- [Poetry](https://python-poetry.org)
- [FastAPI](https://fastapi.tiangolo.com)
- [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/)
- [Amazon SDKs Python Boto3 with Amazon DynamoDB](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/dynamodb.html)
- [React](https://react.dev) (with [PrimeReact](https://primereact.org))
- [Airflow](https://airflow.apache.org)
- [Pytest](https://docs.pytest.org)

## Overview

In this project, we are creating a web app that offers users a unique culinary experience each day. The concept is simple: every day, users will be presented with a single randomly selected recipe from a diverse collection. The daily theme or type of recipe changes, making meal planning more exciting. The concept for each day includes:

- **Monday Dessert**: Start your week with a delightful dessert.
- **Tuesday Breakfast & Brunch**: Enjoy a morning treat to kickstart your day.
- **Wednesday Lunch**: Explore new lunch ideas midweek.
- **Thursday Healthy**: Discover nutritious and wholesome recipes to boost your well-being.
- **Friday Appetizers & Snacks**: Get ready for the weekend with tasty finger foods.
- **Saturday Salads**: Savor refreshing salads for a healthy weekend.
- **Sunday Drinks**: Unwind with a selection of beverages to complete your weekend.

![Become a Head Chef GIF](images/become-a-head-chef.gif)

## Steps followed

### 0. Installation & Setup

- Poetry project

```bash
pip install poetry

# `poetry init --no-interaction` to initialize a pre-existing project
poetry new backend --name="app"
cd backend
poetry add beautifulsoup4 fastapi python-dotenv uvicorn boto3 apache-airflow connexion pytest werkzeug=2.2.3
pip install python-dotenv # to use .env file
# `poetry shell` to access the environment in the terminal and `exit` to exit the environment
```

- React project

```bash
npm create vite@latest frontend -- --template react-ts
cd frontend
npm install --save react react-dom react-router-dom primereact primeicons primeflex @mui/icons-material @mui/material @emotion/styled @emotion/react
```

- AWS:
  - Create an IAM user with the right policies to access the DynamoDB service and permissions to read and write data (use the visual editor or create a custom JSON).
  - Store carefully the access keys, for instance in a `.env` file.
  - Be careful with the [Free Tier](https://aws.amazon.com/pm/dynamodb/), it is easy to exceed the limits.
- Airflow (do not forget to run with Poetry: `poetry run...`).

```bash
# At the backend root
# Everytime for a new terminal, must execute the line below to put correctly the AIRFLOW_HOME
export AIRFLOW_HOME=$(pwd)/airflow
airflow db migrate
# And here replace the generated airflow.cfg by the personal one
cp -fr airflow_files/* airflow/
airflow users create --username admin --firstname admin --lastname admin --role Admin --email admin --password admin

# To see the users
airflow users list

# To run the webserver and the scheduler
airflow webserver -p 8080
airflow scheduler
```

```bash
# Some parameters changed made in the airflow.cfg
# No need to change the default_time, it is utc which is recommended
executor = SequentialExecutor
dags_are_paused_at_creation = False
load_examples = False
expose_config = True
```

- Run the backend and frontend (without Docker)

```bash
# In the backend
poetry run uvicorn app.main:app --reload

# In the frontend (for the development mode)
npm run dev # for the development mode

# In the frontend (to build the production mode)
npm run build

# In the frontend (to test the production mode locally)
npm run preview

# In the frontend (to serve the production mode)
npm install -g serve # if not already installed
serve -s dist
```

### 1. Project architecture

- `backend`: contains the backend code (Python)
  - `app`: contains the code for the API (FastAPI)
    - `database`: contains the code to interact with the database (DynamoDB) and the code for scraping the data (BeautifulSoup)
    - `routers`: contains the code for the routers
- `frontend`: contains the frontend code (React)
  - `src`: contains the code for the frontend
    - `components`: contains the code for the components
    - `pages`: contains the code for the pages

### 2. Create DAG (Directed Acyclic Graph)

To create a pipeline orchestrated by Airflow, we need to create a DAG. The DAG is created in the `airflow\dags` folder. Therefore, the `recipes_dag.py` file must be located in the `airflow\dags` folder. The DAG to get the recipe of the day is scheduled to run every day at 00:00 UTC.

### Interesting points / Issues I encountered

- When we want to do web scraping, we sometimes need to provide a header to the request. Otherwise, the website will not allow us to scrap the information. To do so, we can use the `headers` parameter of the `requests.get()` function. To get a correct `User-Agent` header, we can use the website [https://urlscan.io](https://urlscan.io).
- In React, with the use of useState, we need to be careful about the initial value. It's better to use `null` than `undefined` as initial value.
- Trick for CSS: use the console to see which classe(s) are used for a specific element. Then, we can use the same class in our CSS file.
- Use of Airflow: downgrading the `werkzeug` package to version 2.2.3 to avoid the error `ImportError: from werkzeug.urls import url_decode` when migrating the Airflow database.
- **Harsh case -> Resolved with a custom function.** (in `recipes_dag.py`) Use of Airflow [XComs](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/xcoms.html): a mechanism to let tasks talk to each other, especially with [PythonOperator](https://airflow.apache.org/docs/apache-airflow/2.0.0/howto/operator/python.html#howto-operator-pythonoperator).
- **WARNING for Windows Users**: `pwd` module does not work on Windows as it is a UNIX only package for managing passwords (used to start the airflow server...).

### Extra: Setup of Makefile

```bash
# To run everything
make # or make all

# To run tests
make test
```

### Extra: Setup of pytest

Once the test files are written, we can run the tests.

```bash
pip install pytest

# To run tests
pytest
```

### Extra: Setup of pre-commit

```bash
pip install pre-commit
```

Once the `.pre-commit-config.yaml` completed, we need to set up the git hooks scripts.

```bash
pre-commit install
```
