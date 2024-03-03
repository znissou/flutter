These flex widgets let you create flexible layouts in both the horizontal (Row) and vertical (Column) directions. The design of these objects is based on the web’s flexbox layout model.

## the use of Expanded in Row and Column
`Expanded` Widget expands the child widget to fill any remaining available space that hasn’t been consumed by the other children.

```
Row(
    children: [
        Text(),
        Expanded(child: Text()),
        Text()
    ]
)
```
in this case the seconde text will fill (shrinks) the remaining space between the first and the thrid

You can have multiple Expanded children and determine the ratio in which they consume the available space using the flex argument to Expanded.

```
 child: Row(
        children: <Widget>[
          Expanded(
            flex: 2,
            child: Container(
              color: Colors.amber,
              height: 100,
            ),
          ),
          Container(
            color: Colors.blue,
            height: 100,
            width: 50,
          ),
          Expanded(
            child: Container(
              color: Colors.amber,
              height: 100,
            ),
          ),
        ],
      ),
```