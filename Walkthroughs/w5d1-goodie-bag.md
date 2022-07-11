# [W5D1 Goodie Bag Walkthrough](hhttps://github.com/FullstackAcademy/GoodieBag)
[⬅ Go Back](./walkthrough-directory.md)

## Instructions
- `npm install`
- `createdb goodie-bag`
  - If you check `server/db/database.js` it says `${pkg.name}`
  - Go to `package.json` and look for the name - "goodie-bag"
  - `server/db/database` - add package name to `const db = new Sequelize...`
    ```Javascript
    const db = new Sequelize(`postgres://localhost:5432/${pkg.goodiebag}`, {
      logging: false // so we don't see all the SQL queries getting made
    })
    ```
- `npm run start:dev`


#### Backend

- [X] **Write a `candies` model with the following information:**
  - [X] **name - not empty or null**
  - [X] **description - not empty or null**
  - [X] **quantity - cannot exceed 10 of the same candy in your goodie bag**
  - [X] **imageUrl - with a default value**
  - `server/db/models/candy` - need to define the model `Candy` (use dummy data and requirements as reference)
    - Some need to be [validated](https://sequelize.org/docs/v6/core-concepts/validations-and-constraints/)
  ```Javascript
  const Sequelize = require('sequelize');
  const db = require('../database');

  module.exports = db.define('candy', {
    name: {
      type: Sequelize.STRING,
      allowNull: false,
      validate: {
        notEmpty: true
      }
    },
    description: {
      type: Sequelize.TEXT,
      allowNull: false,
      validate: {
        notEmpty: true
      }
    },
    quantity: {
      type: Sequelize.INTEGER,
      validate: {
        min: 0,
        max: 10
      }
    },
    imageUrl: {
      type: Sequelize.TEXT,
      defaultValue: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTLlHKhK0NMNG3Li4-MogcKD67VvJ3dlq8IZw&usqp=CAU'
    }
  });
  ```
  - `npm run seed`
- [X] **Write a route to serve up all candies**
  - `server/db/index.js`
    - Require `Candy` and export it!
    ```Javascript
    const db = require('./database')
    const Candy = require('../db/models/Candy')

    module.exports = {
      db,
      Candy
    }
    ```
  - `server/api/index.js` - all routes here are already mounted on '/api/
    - Require `Candy` and write the route to get all Candy information
    ```Javascript
    const Candy = require('../db/models/Candy');

    router.get('/candies', async (req, res, next) => {
      try {
        const allCandy = await Candy.findAll()
        res.send(allCandy)
      } catch (err) {
        next(err)
      }
    })
    ```
#### Frontend
- [X] Write a candies sub-reducer to manage candies in your Redux store
  - `app/reducers/index.js`
    - Import axios
      ```Javascript
      import axios from 'axios'
      ```
    - Make an action type, action creator, and thunk. Add the case to the reducer
      - The thunk `fetchCandies` will get the data from '/api/candies' which shows all candies and then dispatch it
      - In `initialState`, make `candies` an empty array
      - In the reducer, `SET_CANDIES` will give the rest of the state + the candies
      ```Javascript
      // Action Type
      const SET_CANDIES = 'SET_CANDIES'

      // Action Creators
      const _setCandies = (candies) => {
        return {
          type: SET_CANDIES,
          candies
        }
      }

      // Thunks
      export const fetchCandies = () => {
        return async (dispatch) => {
          const { data } = await axios.get('/api/candies')
          dispatch(_setCandies(data))
        }
      }

      // Reducer
      const initialState = {
        candies: [],
      }

      const rootReducer = (state = initialState, action) => {
        switch (action.type) {
          case SET_CANDIES:
            return {...state, candies: action.candies}
          default:
            return state
        }
      }
      ```
- [X] Write a component to display a list of all candies
  - Make `AllCandies.js` in `app/components`
  - Import everything that needs to be imported...
    ```Javascript
    import React, { Component } from 'react'
    import { connect } from 'react-redux'
    import { fetchCandies } from '../reducers/index'
    import Candy from '../../server/db/models/Candy'
    ```
  - `mapStateToProps` - make `candies` accessible as a prop to be mapped in render body
    ```Javascript
    const mapStateToProps = (state) => ({
      candies: state.candies
    })
    ```
  - `mapDispatchToProps` - `fetchCandies` to be used in `componentDidMount()`
    ```Javascript
    componentDidMount() {
      this.props.fetchCandies()
    }
    ...
    const mapDispatchToProps = (dispatch) => ({
      fetchCandies: () => dispatch(fetchCandies())
    })

    export default connect(mapStateToProps, mapDispatchToProps)(AllCandies)
    ```
    - `console.log(this.props)` returns {`candies`: [{}, {} ,{}], `fetchCandies`, `history`, `location`, `match`}
- [X] Display the all-candies component when the url matches `/candies`
  - `app/components/root.js`
    - Import `AllCandies` and stuff from 'react-router-dom'
      ```Javascript
      import AllCandies from './AllCandies'
      import {HashRouter as Router, Route, Switch, Link} from 'react-router-dom'
      ```
    - Import `<Route>` for `AllCandies`
      - Wrap the div in `<Router>`
      - Give the `<Route>` an `exact` prop so that it doesn't fuzzy match
      ```Javascript
      const Root = () => {
        return (
          <Router>
            <div>
              <nav>
                Goodie Bag
              </nav>
              <main>
                <h1>Welcome to the Goodie Bag!</h1>
                <p>What a nice home page for your goodies!</p>
                <Route exact path="/candies" component={AllCandies} />
              </main>
            </div>
          </Router>
        )
      }
      ```
  - `app/components/AllCandies.js`
    - .map `this.props.candies` to show each candy
      ```Javascript

      ```
- [ ] Add links to the navbar that can be used to navigate to the all-candies view and the home view (`/`)