---
layout: post
title: "For Loops in React Render() - No You Didn't!"
date:   2017-08-20 12:11:03 +0200
categories: react
---
Let's say you want to programmatically create a table in React. Something like:

``` html
<table>
  <tr className="table-row-1">
    <td>Column 1</td>
    <td>Column 2</td>
    <td>Column 3</td>
    <td>Column 4</td>
    <td>Column 5</td>
  </tr>
  <tr className="table-row-2">
    <td>Column 1</td>
    <td>Column 2</td>
    <td>Column 3</td>
    <td>Column 4</td>
    <td>Column 5</td>
  </tr>
  <tr className="table-row-3">
    <td>Column 1</td>
    <td>Column 2</td>
    <td>Column 3</td>
    <td>Column 4</td>
    <td>Column 5</td>
  </tr>
</table>
```


Looks like we could use for loops to generate the desired code. In this case we'd need a nested loop, one of the three rows and one for each of their five children.

Million Dollar Question:
What happens when you stick the for loops in the render() function?

```
render() {
  return(
    <Table>
      // I am a for loop! Can I stay here?
    </Table>
  )
}
```

Error!
=> "Syntax error: Unexpected token"

![Sassy No You Didnt]({{ site.url }}/assets/170820_no_you_didnt.gif)

Apparently React does not like for loops in its render() method! How do we solve this?

What we need to do is create the table in a separate function outside render(), but still inside the class. Then we can call the creating function inside render to get the result.

But. There are a few pitfalls to look out for. This will NOT work:

```
createTable = () => {
    let table = []

    for (let i = 0; i < 3; i++) {
      table.push(<tr>
        {
          //inner loop to create columns
        }
      </tr>)
    }
    return table
  }
```

Why? You cannot create children for a parent that does not exist yet.
What we need to do in the first step is to create all the children.
And then in the second step we can create the parent (our table row <tr>) and assign the children to it.

Our code will look like this:

{% highlight linenos %}

import React from 'react'

export default class myTable extends React.Component {

  createTable = () => {
    let table = []

    // Outer loop to create parent
    for (let i = 0; i < 3; i++) {
      let children = []
      //Inner loop to create children
      for (let j = 0; j < 5; j++) {
        children.push(<td>{`Column ${j + 1}`}</td>)
      }
      //Create the parent and add the children
      table.push(<tr>{children}</tr>)
    }
    return table
  }


  render() {
    return(
      <table>
        {this.createTable()}
      </table>
    )
  }

}
{% endhighlight %}

And an extra tipp for those who are using Semantic UI: <Table> has property of "children".  So you'd create your row like this:
```
table.push(<Table.Row children={children} />)
```

<br>
For more details, please visit the [sematic react ui documentation][semantic].

[semantic]: https://react.semantic-ui.com/collections/table
