# Joystick-like selection component

## Installation
### npm:
```bash
npm i @mazi1/joystick
```
## Usage
```js
import Joystick from '@mazi1/joystick'
```

```js
<template>
  <Joystick></Joystick>
</template>

<script>
import Joystick from './components/Joystick'

export default {
  name: 'App',
  components: {
    Joystick
  }
}
</script>
```

## Documentation
Use properties to define:
- number of entries/points (number_of_points) 
- color (color) 
- distance from points to joystick (width_points_circle) 
- width of joystick (circle_width)
- font size (font_size)
- properties of entries (entry_props) *must be passed in as array of objects with value and link property (optional)

```js
<template>
  <Joystick 
    :number_of_points="6" 
    :color="'#ff9933'" 
    :width_points_circle="200" 
    :circle_width="50" 
    :font_size="18"
    :entry_props="[
      {value: 'Cute Cat!',
       linkurl: 'https://cdnuploads.aa.com.tr/uploads/Contents/2020/05/14/thumbs_b_c_88bedbc66bb57f0e884555e8250ae5f9.jpg?v=140708' },
      {value: 'Entry 2'},
      {value: 'Entry 3'},
      {value: 'Entry 4'},
      {value: 'Entry 5'},
      {value: 'Entry 6'}
    ]">
  </Joystick>
</template>
```