# pydantic-to-typescript

A simple CLI tool for converting pydantic models into typescript interfaces. Useful for any scenario in which python and javascript applications are interacting, since it allows you to have a single source of truth for type definitions.

This tool requires that you have the lovely json2ts CLI utility installed. Instructions can be found here: https://www.npmjs.com/package/json-schema-to-typescript

### Installation
```bash
$ pip install pydantic-to-typescript
```
---
### CLI

|Prop|Description|
|:----------|:-----------|
|&#8209;&#8209;module|name of the python module you would like to convert. All the pydantic models within it will be converted to typescript interfaces. Discoverable submodules will also be checked. Ex: 'pydantic2ts.examples.pydantic_models'|
|&#8209;&#8209;output|name of the file the typescript definitions should be written to. Ex: './frontend/apiTypes.ts'|
|&#8209;&#8209;json2ts&#8209;cmd|optional, the command used to invoke json2ts. The default is 'json2ts'. Specify this if you have it installed in a strange location and need to provide the exact path (ex: /myproject/node_modules/bin/json2ts)|
---
### Usage
Define your pydantic models (ex: /backend/api.py):
```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import List, Optional

api = FastAPI()

class LoginCredentials(BaseModel):
    username: str
    password: str

class Profile(BaseModel):
    username: str
    age: Optional[int]
    hobbies: List[str]

class LoginResponseData(BaseModel):
    token: str
    profile: Profile

@api.post('/login/', response_model=LoginResponseData)
def login(body: LoginCredentials):
    profile = Profile(**body.dict(), age=72, hobbies=['cats'])
    return LoginResponseData(token='very-secure', profile=profile)
```
Execute the command for converting these models into typescript definitions:
```bash
$ pydantic2ts --module backend.api --output ./frontend/apiTypes.ts
```
The models are now defined in typescript...
```ts
/* tslint:disable */
/**
/* This file was automatically generated from pydantic models by running pydantic2ts.
/* Do not modify it by hand - just update the pydantic models and then re-run the script
*/

export interface LoginCredentials {
  username: string;
  password: string;
}
export interface LoginResponseData {
  token: string;
  profile: Profile;
}
export interface Profile {
  username: string;
  age?: number;
  hobbies: string[];
}
```
...and can be used in your typescript code with complete confidence.
```ts
import { LoginCredentials, LoginResponseData } from './apiTypes.ts';

async function login(
  credentials: LoginCredentials,
  resolve: (data: LoginResponseData) => void,
  reject: (error: string) => void,
) {
  try {
    const response: Response = await fetch('/login/', {
      method: 'POST',
      headers: {'Content-Type': 'application/json'},
      body: JSON.stringify(credentials),
    });
    const data: LoginResponseData = await response.json();
    resolve(data);
  } catch (error) {
    reject(error.message);
  }
}
```