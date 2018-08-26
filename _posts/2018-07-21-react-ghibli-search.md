---
layout: post
title: "Studio Ghibli - movies app"
author: "Predrag Stošić"
photo_url: "/assets/ghibli-search/ghibli-search.png"
---

> Notice: This article is not a tutorial, but just a simple and quick overview on how this app was built.

GitHub repository [public-apis](https://github.com/toddmotto/public-apis) is a great place if you're looking for a free API for your next project. One of them is Studio's Ghibli API which contains a list of their movies, as well as a list of characters, places etc. Since I'm learning React, I decided to try it out in a mini project, simply called Ghibli Search. In this article, I'll show you how I made this simple application. Code in this article will be simplified (without props, classes etc.).

### Dependencies

Before we begin, here is a list of dependencies I used to build this app:

- [Reactstrap](https://reactstrap.github.io/)
- [FontAwesome](https://fontawesome.com/how-to-use/on-the-web/using-with/react)
- [Immutability-helper](https://github.com/kolodny/immutability-helper)

You can install each of them like this:

- Reactstrap (check their website for full explanation on how to implement it)

```sh
npm install --save reactstrap react react-dom
```

- FontAwesome

```sh
npm i --save @fortawesome/fontawesome-svg-core
npm i --save @fortawesome/free-solid-svg-icons
npm i --save @fortawesome/react-fontawesome
```

- Immutability-helper

```sh
npm install immutability-helper --save
```

### Breaking the UI into components

Since React is built around components, the first thing we want to do when creating an app is to break it's UI into components.

![Ghibli Search](/assets/ghibli-search/ghibli-search-components.png)

According to this, our components are as following:

- Main app
- Searchbar
- Sidebar
- Movie

This layout could be broken into more simpler components, but for the sake of simplicity we'll keep them like this.

### Main component

Our app component, stripped of all styling and classes, should look like this:

```javascript
return (
  <div>
    <SearchBar />
    <Container>
      <Row>
        <Collapse>
          <Nav>
            <Sidebar />
          </Nav>
        </Collapse>
        <Col>
          <Row>{main}</Row>
          {loading}
        </Col>
      </Row>
    </Container>
  </div>
);
```

As you can see, it simply renders SearchBar and Sidebar component including Bootstrap layout. Our movie component is rendered conditionaly. What component will render inside _{main}_ dependens on what element is selected in sidebar. I'll explain how this is implemented in the next step.

### Fetching data

Since the main, or landing page is displaying a list of movies we need to fetch information about them from Ghibli API as soon as component renders. We could do this using default _fetch()_ function or some third party library like _axios_. For doing any action after component renders we can use _componentDidMount()_ lifecycle method which is invoked immediately after a component is mounted. You can find more about lifecycle methods in React [here](https://reactjs.org/docs/react-component.html). Here is an example of how it can be used in practice for fetching list of movies:

```javascript
fetchPeople = () => {
    const url = "https://ghibliapi.herokuapp.com/films";
    fetch(people)
      .then(res => res.json())
      .then(data =>
        this.setState({ movies: data })
      );
  }
};
```

### Conditionaly rendering components

Since we have multiple choices in the side bar we have to find a way to rerender our main component with the selected one. This could be done with React's router, but for the sake of simplicity I did this with conditionaly rendering. Selected menu is stored in application state. We simply check what is current selected menu and assing corresponding component to _main_ variable, like this:

```javascript
if (menu === "all") {
  main = this.state.movies.map(movie => {
    return <Movie />;
  });
} else if (menu === "people") {
  main = <Person />;
} else if (menu === "locations") {
  main = <Location />;
}
```

### Movie component

This component is a home for single movie information like title, poster image, score etc. For styling I used Bootstrap cards. Our movie component initialy shows poster, Rotten Tomatoes score, title, year and director. But when we click on it, popover is rendered (also from Bootstrap) with added description and producer. This is the simplified layout of this component:

```javascript
return (
  <Col>
    <Card>
      <CardImg />
      <Badge>{score}</Badge>
      <CardText>{title}</CardText>
      <CardText>
        {releaseDate}, {director}
      </CardText>
    </Card>
    <Popover>
      <PopoverBody>
        <p>{title}</p>
        <p>{releaseDate}</p>
        <p>{description}</p>
        <p>
          <b>Director:</b> {director}
        </p>
        <p>
          <b>Producer:</b> {producer}
        </p>
      </PopoverBody>
    </Popover>
  </Col>
);
```

### Handling user input and filtering data

The next thing we need to do is handle user input and filter movie list according to input value. To achieve this we need to do two things:

- Listen for user inputs with the _onChange_ event listener, so that we can trigger the onSearch method
- Set the value of the input field explicitly using _this.state.search_

After _search_ property in the _state_ is changed we use that value to filter our movie list and display rerendered list with only those movies. To filter the movies we need to do the following:

- First, we need the copy of _movies_ array so that the old one won't be erased while filtering
- Second, we need to run regular expression on movie titles using _filter()_ function like this:

```javascript
this.state.filterMovies.filter(movie =>
  new RegExp(e.target.value, "i").exec(movie.title)
);
```

Here we are executing regular expresion on _movie.title_ and comparing it with our user input (_e.target.value_).

### Sorting data

In app's sidebar we have an option to sort our data in following ways:

- Year ascending (default)
- Year descending
- Rating ascending
- Rating descending

To get the selected choice we can create _handleChecked()_ function which takes _choice_ as an argument. _Choice_ argument is provided by our sidebar component when we call _handleChecked()_ function. After our choice is received we can store it in our state and then sort the movies. To sort the movies we can use _sort()_ function:

```javascript
sortedMovies.sort((a, b) => a.release_date - b.release_date);
```

Here, we are sorting our movies by year ascending. You can find more about _sort()_ function [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort) .

### Conclusion

This is very a simple example on how to use an API with React. As I said at the beginning, this is not a tutorial, rather just an overview of this app. If you want to learn more about React, check the official React [documentation](https://reactjs.org/docs/getting-started.html). Thanks for reading and happy coding.
