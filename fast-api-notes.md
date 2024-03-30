# DB and global settings
- Setting default timezone for instance of app
  - ```python
      import os, time
      
      os.environ['TZ'] = 'Asia/Tashkent'  # or make it variable
      time.tzset()
    ```
- Model creation
  - On creating model, set all datetime fields as timezone=False, to set it not aware of timezone, 'cause SQL will set UTC by default, we are setting it into the main file.

# Alembic migrations
Alembic should be inited when you first time use it (here is already initiated):
```bash
    alembic init alembic
```
To create alter migration:
```bash
    alembic revision -m "alter <table_name> table"
```
To run migration (should be run from inside the container, please, check Makefile for example):
```bash
    alembic upgrade head
```

# Query parameters

- Multiple query params:
  - To receive it, you should use list Query param, you need to use one of int | str | bool for typing: 
  - ```python
    async def func(
        param_name: list[int | str | bool] | None = Query(None),
    ):
      ...
    ```
  - On the front-end side you need to clear all brackets [] from the query, which you build (it will automatically set by default for multiple same keys)
  - For that, you need to install qs lib. ```npm install qs```
    - ```js
      import axios from 'axios';
      import qs from 'qs'
      
      class SomeClassService{
        get_list(list_param){
            let params = {
                list_param,
            }
            let axiosMultipleQueryValues = axios.create({
                paramsSerializer: params => qs.stringify(params, {arrayFormat: 'repeat'})  // here are we serialize array params (list)
            })
            return axiosMultipleQueryValues.get('/somewhere', {params}).then(response => response)
        }
      }
      ```

# Example project: <a href="">Backend</a>