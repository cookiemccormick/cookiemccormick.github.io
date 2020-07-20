---
layout: post
title:      "Scout - React Redux Project"
date:       2020-07-20 02:37:29 +0000
permalink:  scout_-_react_redux_project
---


Finally, the final project!!! 

![](https://i.imgur.com/MXpQoTb.gif)

I took some time off after the Rails Javascript Project.  I passed the assessment and then had a baby a few hours later.  Life has been pretty busy since and it's been hard finding the time to code.  So, I'm really glad to be able to finish all assignments and projects. 

For my React Redux final project, I created a CRUD app called Scout, which lists models and their bookings.  This app is the beginning of what I would have liked in my previous position as Talent Scheduling Specialist at HSN.  I was responsible for finding models to demo products on TV.  I didn't have any sort of special app to help with my daily duties.  Model pictures were in folders on my desktop.  Bookings were done on spreadsheets and emailed.  I had comp cards in a folder.  It would've been nice the information in one place. 

## Scout
Scout is a database for models, their stats and bookings.  

![](https://i.imgur.com/mLdrOtq.png)

### Backend
Scout uses an Rails API backend to handle the data persistence with the database.  Two models were created, a model `model` (sounds confusing) and a booking `model`.  

```
class Model < ApplicationRecord
  has_many :bookings
  has_one_attached :picture

  validates :name, presence: true
  validates :height, presence: true
  validates :bust, presence: true, inclusion: { in: 20..50, message: 'must be 20-50' }
  validates :waist, presence: true, inclusion: { in: 20..50, message: 'must be 20-50' }
  validates :hip, presence: true,  inclusion: { in: 20..50, message: 'must be 20-50' }
  validates :shoe, presence: true, inclusion: { in: 5..12, message: 'must be 5-12' }
  validates :eyes, presence: true
  validates :hair, presence: true

  scope :alphabetical_name, -> { order("name asc") }
end
```
```
class Booking < ApplicationRecord
  belongs_to :model

  validates :model_id, presence: true
  validates :job, presence: true
  validates :amount, presence: true
  validates :start_time, presence: true
  validates :end_time, presence: true
  validates :description, presence: true

  scope :most_recent, -> { order("start_time desc") }
end
```

Since bookings are all associated with a model, routes and serializers are nested:

```
Rails.application.routes.draw do
  default_url_options host: "localhost", port: 3000

  namespace :api do
    namespace :v1 do
      resources :models do
        resources :bookings
      end
    end
  end
end
```

```
class ModelSerializer < ActiveModel::Serializer
  include Rails.application.routes.url_helpers

  attributes :id, :picture, :name, :height, :bust, :waist, :hip, :shoe, :eyes, :hair, :bookings

  has_many :bookings

  def picture
    return unless object.picture.attached?
    url_for(object.picture)
  end

  def bookings
    object.bookings.most_recent
  end
end
```
```
class BookingSerializer < ActiveModel::Serializer
  attributes :id, :job, :amount, :start_time, :end_time, :description, :model_id

  belongs_to :model

  def start_time
    object.start_time.strftime('%FT%R')
  end

  def end_time
    object.end_time.strftime('%FT%R')
  end
end
```

### Frontend
I used Create React App to setup the frontend project.  For styling, I used React Bootstrap.  

The model form is used for both created a new model and updating a model. Below is the example of how a model is created:

```
import React from 'react';
import Form from 'react-bootstrap/Form';
import Button from 'react-bootstrap/Button';

const HEIGHTS = [
  '5\'7"',
  '5\'8"',
  '5\'9"',
  '5\'10"',
  '5\'11"',
  '6\'',
  '6\'1"',
  '6\'2"',
  '6\'3"',
  '6\'4"',
  '6\'5"'
];

class ModelForm extends React.Component {
  state = {
    name: this.props.model.name || '',
    picture: '',
    height: this.props.model.height || '',
    bust: this.props.model.bust || '',
    waist: this.props.model.waist || '',
    hip: this.props.model.hip || '',
    shoe: this.props.model.shoe || '',
    eyes: this.props.model.eyes || '',
    hair: this.props.model.hair || ''
  };

  handleChange = (event) => {
    this.setState({
      [event.target.name]: event.target.value
    });
  }

  handleOnSubmit = (event) => {
    event.preventDefault();

    const model = {...this.state, id: this.props.model.id};
    const formData = new FormData(event.target);

    this.props.onSubmit(formData, model);
  }

  render() {
    return (
      <div>
        <Form onSubmit={this.handleOnSubmit} encType="multipart/form-data">
          <Form.Group controlId="formName">
            <Form.Label>Model Name:</Form.Label>
            <Form.Control type="text" placeholder="Enter name" name="name" onChange={this.handleChange} value={this.state.name} required/>
          </Form.Group>
          <Form.Group controlId="formPicture">
            <Form.Label>Picture:</Form.Label>
            <Form.Control type="file" name="picture" onChange={this.handleChange}/>
          </Form.Group>
          <Form.Group controlId="formHeight">
            <Form.Label>Height:</Form.Label>
            <Form.Control as="select" value={this.state.height} onChange={this.handleChange} name='height'>
              {HEIGHTS.map(value => {
                return (
                  <option key={value}>{value}</option>
                );
              })}
            </Form.Control>
          </Form.Group>
          <Form.Group controlId="formBust">
            <Form.Label>Bust:</Form.Label>
            <Form.Control type="number" placeholder="Enter Bust" name="bust" onChange={this.handleChange} value={this.state.bust} required min="20" max="50"/>
          </Form.Group>
          <Form.Group controlId="formWaist">
            <Form.Label>Waist:</Form.Label>
            <Form.Control type="number" placeholder="Enter Waist" name="waist" onChange={this.handleChange} value={this.state.waist} required min="20" max="50"/>
          </Form.Group>
          <Form.Group controlId="formHip">
            <Form.Label>Hip:</Form.Label>
            <Form.Control type="number" placeholder="Enter Hip" name="hip" onChange={this.handleChange} value={this.state.hip} required min="20" max="50"/>
          </Form.Group>
          <Form.Group controlId="formHip">
            <Form.Label>Shoe:</Form.Label>
            <Form.Control type="number" placeholder="Enter Shoe" name="shoe" onChange={this.handleChange} value={this.state.shoe} required min="5" max="12"/>
          </Form.Group>
          <Form.Group controlId="formEyes">
            <Form.Label>Eyes:</Form.Label>
            <Form.Control type="text" placeholder="Enter Eyes" name="eyes" onChange={this.handleChange} value={this.state.eyes} required/>
          </Form.Group>
          <Form.Group controlId="formHair">
            <Form.Label>Hair:</Form.Label>
            <Form.Control type="text" placeholder="Enter Hair" name="hair" onChange={this.handleChange} value={this.state.hair}/>
          </Form.Group>
          <Button variant="primary" type="submit">
            Submit
          </Button>
        </Form><br/>
      </div>
    );
  }
}

export default ModelForm;
```

`mapStateToProps` is used to connect the application’s state to the model prop.

```
import React from 'react';
import { connect } from 'react-redux';
import { Redirect } from 'react-router-dom';

import { addModel } from '../actions/addModel';
import ModelForm from './ModelForm';

class ModelInput extends React.Component {
  handleOnSubmit = (formData, model) => {
    this.props.addModel(formData, model);
  }

  render() {
    //after model is created, redirect to model show page
    if (this.props.model) {
      return <Redirect to={`/models/${this.props.model.id}`}/>;
    }

    return (
      <div>
        <h4>Create Model:</h4>
        <ModelForm onSubmit={this.handleOnSubmit} model={{}} />
      </div>
    );
  }
}

const mapStateToProps = state => {
  return {
    model: state.model
  };
}

export default connect(mapStateToProps, { addModel })(ModelInput);
```

The action makes a `fetch()` `POST` request to the RAILS API and converts to JSON.  The action creator is dispatched.

```
export const addModel = (formData, data) => {
  return (dispatch) => {
    fetch('http://localhost:3000/api/v1/models', {
      method: 'POST',
      headers: {
        'Accept': 'application/json'
      },
      body: formData
    })
    .then(response => response.json())
    .then(model => dispatch({
      type: 'ADD_MODEL',
      payload: model
    }))
  };
}
```

The reducer returns the updated state.

```
function manageModels(state = {models: []}, action) {
  switch (action.type) {
    case 'ADD_MODEL':
      return {model: action.payload};
    default:
      return state;
  }
}

export default manageModels;
```

Feel free to check out the application.

**React Redux Application Link:**

https://github.com/cookiemccormick/scout
