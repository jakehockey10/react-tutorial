# Comment

```javascript
var converter = new Showdown.converter();
var Comment = React.createClass({
  render: function() {
    var rawMarkup = converter.makeHtml(this.props.children.toString());
    return (
      <div className="comment">
        <h2 className="commentAuthor">
          {this.props.author}
        </h2>
        <span dangerouslySetInnerHTML={{__html: rawMarkup}} />
      </div>
    );
  }
});
```
### Q1: Where does the value of ``this.props.author`` get specified?

{{ this.props.author is specified in the declaration of commentNodes.  Author is an attribute of the HTML like node in the render function of CommentList }}

### Q2: Where does the value of ``this.props.children`` get specified?

{{ this.props.children consists of all the child elements of the comments that are passed into CommentList }}

### Q3: What does ``className="comment"`` do?

{{ this gives the rendered HTML element a class attribute with value 'comment' }}

### Q4: What is ``dangerouslySetInnerHTML``? Why is it such a long word for an API method?

{{ dangerouslySetInnerHTML allows for React to ignore the danger of an XSS attack and allow HTML to be rendered using Showdown. }}

# CommentBox
```javascript
var CommentBox = React.createClass({
  loadCommentsFromServer: function() {
    $.ajax({
      url: this.props.url,
      dataType: 'json',
      success: function(data) {
        this.setState({data: data});
      }.bind(this),
      error: function(xhr, status, err) {
        console.error(this.props.url, status, err.toString());
      }.bind(this)
    });
  },

```

### Q5: How does ``$`` get defined? Is it part of the ReactJS framework?

{{ The '$' comes from JQuery.  It is not a part of the ReactJS framework. }}

### Q6: Where does the value of ``this.props.url`` get specified?

{{ this.props.url is specified in our call to React.render.  We specify our CommentBox to have a url property with value 'comments.json'. }}

### Q7: What would happen to the statement ``this.setState`` if ``bind(this)`` is removed? Why?

{{ Without bind(this), this.setState would refer to the setState property of the wrong object (in this case $). }}

### Q8: Who calls ``loadCommentsFromServer``? When? 

{{ The componentDidMount property function of CommentBox calls loadCommentsFromServer once upon load and then we set an interval to call this method on a regular basis. }}


```javascript
	
  handleCommentSubmit: function(comment) {
    var comments = this.state.data;
    comments.push(comment);
    this.setState({data: comments}, function() {
      // `setState` accepts a callback. To avoid (improbable) race condition,
      // `we'll send the ajax request right after we optimistically set the new
      // `state.
      $.ajax({
        url: this.props.url,
        dataType: 'json',
        type: 'POST',
        data: comment,
        success: function(data) {
          this.setState({data: data});
        }.bind(this),
        error: function(xhr, status, err) {
          console.error(this.props.url, status, err.toString());
        }.bind(this)
      });
    });
  },
  getInitialState: function() {
    return {data: []};
  },
  componentDidMount: function() {
    this.loadCommentsFromServer();
    setInterval(this.loadCommentsFromServer, this.props.pollInterval);
  },
  render: function() {
    return (
      <div className="commentBox">
        <h1>Comments</h1>
        <CommentList data={this.state.data} />
        <CommentForm onCommentSubmit={this.handleCommentSubmit} />
      </div>
    );
  }
});
```

### Q9: What is the purpose of ``this.state``? How is this different from ``this.props``?

{{ this.state is the state of a particular component.  this.props is immutable and owned by the parent, whereas this.state is mutable and is private to its component.  It is set with this.setState. }}

### Q10: What is the initial value of ``this.state.data``? How is the initial value specified?

{{ this.state.data is initialized in the getInitialState property function.  This function is called exactly once during the lifecycle of the component. }}

### Q11: What is the new value of ``this.state.data``? How is this new value set?

{{ React automatically calls componentDidMount once the component is rendered.  This, in turn, calls loadCommentsFromServer }}

### Q12: What is the purpose of ``componentDidMount`` callback?

{{ React automatically calls this property function once the component is rendered. }}

### Q13: What is the purpose of ``getInitialState``?

{{ getInitialState is called once during the lifecycle of a component is initializes the state of the component. }}

# CommentList

```javascript

var CommentList = React.createClass({
  render: function() {
    var commentNodes = this.props.data.map(function(comment, index) {
      return (
        // `key` is a React-specific concept and is not mandatory for the
        // purpose of this tutorial. if you're curious, see more here:
        // http://facebook.github.io/react/docs/multiple-components.html#dynamic-children
        <Comment author={comment.author} key={index}>
          {comment.text}
        </Comment>
      );
    });
    return (
      <div className="commentList">
        {commentNodes}
      </div>
    );
  }
});
```
### Q14: How does the value of ``this.props.data`` get set?

{{ this.props is passed from parent to child, so this.props.data is set in CommentBox, the parent of CommentList. }}

### Q15: What is the value of ``commentNodes``?

{{ commentNodes is a variable defined by the result of calling the map function on this.props.data and creating a Comment component for each element in this.props.data. }}

### Q16: Where does the value of ``{comment.text}`` go on the rendered page?

{{ comment.text is the text part of a comment in the json data, and will go inside the component/dom element. }}

# CommentForm
```javascript

var CommentForm = React.createClass({
  handleSubmit: function(e) {
    e.preventDefault();
    var author = this.refs.author.getDOMNode().value.trim();
    var text = this.refs.text.getDOMNode().value.trim();
    if (!text || !author) {
      return;
    }
    this.props.onCommentSubmit({author: author, text: text});
    this.refs.author.getDOMNode().value = '';
    this.refs.text.getDOMNode().value = '';
  },
  render: function() {
    return (
      <form className="commentForm" onSubmit={this.handleSubmit}>
        <input type="text" placeholder="Your name" ref="author" />
        <input type="text" placeholder="Say something..." ref="text" />
        <input type="submit" value="Post" />
      </form>
    );
  }
});

React.render(
  <CommentBox url="comments.json" pollInterval={2000} />,
  document.getElementById('content')
);
```

### Q17: What is the purpose of ``e.preventDefault()``?

{{ e.preventDefault() prevents the browser from reacting to a submission the way it is expected to by default.  We are basically telling it to let us handle form submission. }}

### Q18: What is the value of ``this.props.onCommentSubmit``? What does it get specified?

{{ this.props.onCommentSubmit is bound to the handleCommentSubmit property function of the CommentBox component. }}

### Q19: Where does ``this.refs.author`` point to?

{{ this.refs points to the component, so this.refs.author refers to the components author property/attribute }}

### Q20: What does ``getDOMNode()`` do?

{{ getDOMNode returns the DOM element that is the component you are calling this method on. }}