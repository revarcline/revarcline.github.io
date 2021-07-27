[Bootstrap](https://getbootstrap.com/) and lightweight alternatives like [minstyle.io](https://minstyle.io/) help designers and developers build out flexible responsive web pages by creating a 12-column grid with css selectors like so: 

```html
<div class="container">
  <div class="row">
    <div class="col-5">5/12 of container</div>
    <div class="col-3">1/4 of container</div>
    <div class="col">flexible (remaining 1/3 of container)</div>
  </div>
  <div class="row">
    <div class="col">single column below</div>
  </div>
</div>
```
<!--more-->

For more information on this system, see [the bootstrap docs](https://getbootstrap.com/docs/4.5/layout/grid/). 

In my latest project, [exquisite](https://exquisite.buckar.ooo), i found myself needing to programatically generate a bootstrap grid for multiple views in a single-page application. In order to most efficiently do this without relying on an existing framework, I created an es6 class called `GridBuilder` which would enable me to clear and generate new scaffolds for content. Here's the class' code:

```javascript
class GridBuilder {
  // create rows and columns in the main container, or within any div
  constructor(container) {
    this.container = container;
  }

  clearContainer() {
    while (this.container.firstChild) {
      this.container.removeChild(this.container.firstChild);
    }
  }

  buildRow(id, valign = "") {
    const row = document.createElement("div");
    row.className = valign === "" ? "row" : `row justify-content-${valign}`;
    row.id = id;
    this.container.appendChild(row);
    return row;
  }

  buildCol(row, id, size) {
    const col = document.createElement("div");
    col.className = size ? `col-${size}` : "col";
    col.id = id;
    row.appendChild(col);
    return col;
  }

  resetGrid() {
    this.clearContainer();
    this.row1 = this.buildRow("top-row", "center");
  }
}
```

With this class, it was a simple matter of creating a new `GridBuilder` object, resetting the grid, and adding the row(s) and columns i need. In practice, i find it convenient to assign them within the gridbuilder object i just created. Here's an example from my code:

```javascript
  const grid = new GridBuilder(container);
  grid.resetGrid();
  grid.col1 = grid.buildCol(grid.row1, "preview-col", 4);
  grid.col2 = grid.buildCol(grid.row1, "entry-col", 4);
```

All I had to do after that was point to `grid.col1` or `grid.col2` to build out the view. The `resetGrid()` method came in especially handy when switching between things, allowing me to wipe the container and start with a fresh top row.
