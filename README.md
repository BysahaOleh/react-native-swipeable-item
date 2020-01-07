# React Native Swipeable Item

A swipeable component with underlay for React Native.<br />
Fully native interactions powered by [Reanimated](https://github.com/kmagiera/react-native-reanimated) and [React Native Gesture Handler](https://github.com/kmagiera/react-native-gesture-handler)

Compatible with [React Native Draggable Flatlist](https://github.com/computerjazz/react-native-draggable-flatlist)

![Swipeable Item demo](https://i.imgur.com/zc2IrRl.gif)

## Install
1. Follow installation instructions for [reanimated](https://github.com/kmagiera/react-native-reanimated) and [react-native-gesture-handler](https://github.com/kmagiera/react-native-gesture-handler)
2. `npm install` or `yarn add` `react-native-swipeable-item` 
3. `import SwipeRow from 'react-native-swipeable-item'`  

### Props

_NOTE:_ Naming is hard. When you swipe _right_, you reveal the item on the _left_. So what do you name these things? I have decided to name everything according to swipe direction. Therefore, a swipe left reveals the `renderUnderlayLeft()` component with width `underlayWidthLeft`. Not perfect but it works.

Name | Type | Description
:--- | :--- | :---
`renderUnderlayLeft` | `(item: T) => React.ReactNode` |  Component to be rendered underneath row on left swipe.
`renderUnderlayRight` | `(item: T) => React.ReactNode` |  Component to be rendered underneath row on left swipe.
`underlayWidthLeft` | `number` | Width of left-swiped underlay.
`underlayWidthRight` | `number` | Width of left-swiped underlay.
`onChange` | `(params: { open: "left" \| "right" \| "null" }) => void` |  Called when row is opened or closed.

### Instance Methods
Name | Type | Description
:--- | :--- | :---
`open` | `("left" \| "right") => void` |  Programmatically open left or right.
`close` | `() => void` | Close all.

```js
// Programmatic open example
const itemRef: SwipeRow | null = null

...

<SwipeRow ref={ref => itemRef = ref} />

...
if (itemRef) itemRef.open("left")
```

### Example
```javascript
import React from 'react';
import {
  Text,
  View,
  StyleSheet,
  FlatList,
  LayoutAnimation,
  TouchableOpacity,
} from 'react-native';
import SwipeRow from 'react-native-swipeable-item';
import DraggableFlatList from 'react-native-draggable-flatlist';

const NUM_ITEMS = 10;
function getColor(i) {
  const multiplier = 255 / (NUM_ITEMS - 1);
  const colorVal = i * multiplier;
  return `rgb(${colorVal}, ${Math.abs(128 - colorVal)}, ${255 - colorVal})`;
}

const initialData = [...Array(NUM_ITEMS)].fill(0).map((d, index) => ({
  text: `Row ${index}`,
  key: `key-${index}`, // Note: It's bad practice to use index as your key. Don't do it in production!
  backgroundColor: getColor(index),
}));

class App extends React.Component {
  state = {
    data: initialData,
  };

  deleteItem = item => {
    const updatedData = this.state.data.filter(d => d !== item);
    // Animate list to close gap when item is deleted
    LayoutAnimation.configureNext(LayoutAnimation.Presets.spring);
    this.setState({ data: updatedData });
  };

  itemRefs = new Map();

  renderUnderlayLeft = item => (
    <TouchableOpacity
      onPressOut={() => this.deleteItem(item)}
      style={[styles.row, {
        flex: 1,
        backgroundColor: 'tomato',
        justifyContent: 'flex-end',
      }]}>
      <Text style={styles.text}>{`[x]`}</Text>
    </TouchableOpacity>
  );

  renderUnderlayRight = item => (
    <TouchableOpacity
      style={[styles.row, {
        flex: 1,
        backgroundColor: 'teal',
        justifyContent: 'flex-start',
      }]}
      onPressOut={() => {
        const ref = this.itemRefs.get(item.key);
        if (ref) ref.close();
      }}>
      <Text style={styles.text}>CLOSE</Text>
    </TouchableOpacity>
  );

  renderItem = ({ item, index, drag }) => (
    <SwipeRow
      key={item.key}
      item={item}
      ref={ref => {
        if (ref) this.itemRefs.set(item.key, ref);
      }}
      renderUnderlayLeft={this.renderUnderlayLeft}
      renderUnderlayRight={this.renderUnderlayRight}
      underlayWidthLeft={100}
      underlayWidthRight={200}>
      <View style={[styles.row,  { backgroundColor: item.backgroundColor }]}>
        <TouchableOpacity onLongPress={drag}>
          <Text
            style={styles.text}>
            {item.text}
          </Text>
        </TouchableOpacity>
      </View>
    </SwipeRow>
  );

  render() {
    return (
      <View style={styles.container}>
        <DraggableFlatList
          keyExtractor={item => item.key}
          data={this.state.data}
          renderItem={this.renderItem}
          onDragEnd={({ data }) => this.setState({ data })}
        />
      </View>
    );
  }
}

export default App;

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  row: {
    flexDirection: 'row',
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    padding: 15,
  },
  text: {
    fontWeight: 'bold',
    color: 'white',
    fontSize: 32,
  },
});
```
