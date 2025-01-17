```eval_rst
.. include:: /header.rst 
:github_url: |github_link_base|/widgets/extra/chart.md
```
# Chart (lv_chart)

## Overview

Charts are a basic object to visualize data points. 
They support *Line* charts (connect points with lines and/or draw points on them) and *Bar* charts.

Charts also support:
- division lines
- 2 y axis
- axis ticks and texts on ticks
- cursors
- scrolling and zooming

## Parts and Styles
- `LV_PART_MAIN` The background of the chart. It uses all the typical background related style properties and *line* properties for the division lines. *Padding* makes the series area smaller. 
- `LV_PART_SCROLLBAR` The scrollbar used if the chart is zoomed. See the [Base object](/widgets/obj)'s documentation for details.
- `LV_PART_ITEMS` Refers to the line or bar series.
    - Line chart: The *line* properties are used by the lines. `width`, `height`, `bg_color` and `radius` is used to set the appearance of points.
    - Bar chart: The typical background properties are used to style the bars. 
- `LV_PART_CURSOR` *Line* properties are used to style the cursors.  `width`, `height`, `bg_color` and `radius` is used to set the appearance of points.
- `LV_PART_TICKS` *Line* and *Text* style properties are used to style the ticks

## Usage

### Data series
You can add any number of series to the charts by `lv_chart_add_series(chart, color, LV_CHART_AXIS_PRIMARY/SECONDARY)`.
It allocates data for a `lv_chart_series_t` structure which contains the chosen `color` and an array for the data points.

`LV_CHART_AXIS_PRIMARY/SECONDARY` tells series should use the range of the `LV_CHART_AXIS_PRIMARY` (left) or `LV_CHART_AXIS_SECONDARY` (right) Y axis.

With `lv_chart_set_ext_array(chart, ser, value_array)` makes the chart use an external array for the given series. 
`value_array` should look like this: `lv_coord_t * value_array[num_points]`. The array size needs to be large enough to hold all the points of that series.
The array's pointer will be saved in the chart so it needs to be global, static or dynamically allocated. 
Note: you should call `lv_chart_refresh(chart)` after the external data source has been updated, to update the chart.

The value array of a series can be get by `lv_chart_get_array(chart, ser)`. It can be use with `ext_array` or *normal array*s.

### Series' type
The following data display types exist:
- `LV_CHART_TYPE_NONE`  Do not display any data. It can be used to hide the series.
- `LV_CHART_TYPE_LINE`  Draw lines between the data points and/or points (rectangles or circles) on the data points.
- `LV_CHART_TYPE_BAR` - Draw bars.

You can specify the display type with `lv_chart_set_type(chart, LV_CHART_TYPE_...)`. The types can be 'OR'ed (like `LV_CHART_TYPE_LINE`).

### Modify the data
You have several options to set the data of series:
1. Set the values manually in the array like `ser1->points[3] = 7` and refresh the chart with `lv_chart_refresh(chart)`.
2. Use `lv_chart_set_value_by_id(chart, ser, value, id)` where `id` is the index of the point you wish to update.
3. Use the `lv_chart_set_next_value(chart, ser, value)`.
4. Initialize all points to a given value with: `lv_chart_set_all_value(chart, ser, value)`.

Use `LV_CHART_POINT_DEF` as value to make the library skip drawing that point, column, or line segment.

### Update modes
`lv_chart_set_next_value` can behave in two ways depending on *update mode*:
- `LV_CHART_UPDATE_MODE_SHIFT` Shift old data to the left and add the new one to the right.
- `LV_CHART_UPDATE_MODE_CIRCULAR` - Circularly add the new data (Like an ECG diagram).

The update mode can be changed with `lv_chart_set_update_mode(chart, LV_CHART_UPDATE_MODE_...)`.

### Number of points
The number of points in the series can be modified by `lv_chart_set_point_count(chart, point_num)`. The default value is 10.
Note: this also affects the number of points processed when an external buffer is assigned to a series. So you need to be sure the external array is large enough. 

#### Handling large number of points
On line charts if the number of points is greater than the pixels horizontally, the Chart will draw only vertical lines to make the drawing of large amount of data effective. 
If there are let's say 10 point a pixel, LVGL searches the smallest and the greatest value and draws a vertical lines between them. It ensures to not miss any peaks.

### Vertical range
You can specify the minimum and maximum values in y-direction with `lv_chart_set_range(chart, axis, min, max)`. 
`axis` can be `LV_CHART_AXIS_PRIMARY` (left axis) or `LV_CHART_AXIS_SECONDARY` (right axis).

The value of the points will be scaled proportionally. The default range is: 0..100.

### Division lines
The number of horizontal and vertical division lines can be modified by `lv_chart_set_div_line_count(chart, hdiv_num, vdiv_num)`. 
The default settings are 3 horizontal and 5 vertical division lines. 
If the there is visible border on a side and there is not padding on that side the division line would be drawn on top the border, therefore  on it won't be drawn. 

### Override default start point for series
If you wish a plot to start from a point other than the default which is `point[0]` of the series, you can set an alternative
index with the function `lv_chart_set_x_start_point(chart, ser, id)` where `id` is the new index position to start plotting from.

Note that `LV_CHART_UPDATE_MODE_SHIFT` also changes the `start_point`.

### Tick marks and labels
Ticks and labels can be added to the axis with `lv_chart_set_axis_tick(chart, axis, major_len, minor_len, major_cnt, minor_cnt, label_en, draw_size)`.
- `axis` can be `LV_CHART_AXIS_X/PRIMARY_Y/SECONDARY_Y`
- `major_len` is the length of major ticks
- `minor_len` is the length of minor ticks
- `major_cnt` is the number of major ticks on the axis
- `minor_cnt` in the number of minor ticks between two major ticks
- `label_en` `true`: enable label drawing on major ticks
- `draw_size` extra size required to draw the tick and labels (start with 20 px and increase if the ticks/labels are clipped)

### Zoom
The chart can be zoom independently in x and y directions with `lv_chart_set_zoom_x(chart, factor)` and `lv_chart_set_zoom_y(chart, factor)`.
If `factor` is 256 there is no zoom. 512 means double zoom, etc. Fractional values are also possible but &lt; 256 value is not allowed.


### Cursor

A cursor can be added with `lv_chart_cursor_t * c1 = lv_chart_add_cursor(chart, color, dir);`. 
The possible values of `dir`  `LV_DIR_NONE/RIGHT/UP/LEFT/DOWN/HOR/VER/ALL` or their OR-ed values to tell in which direction(s) should the cursor be drawn.  

`lv_chart_set_cursor_pos(chart, cursor, &point)` sets the position of the cursor. 
`pos` is a pointer to an `lv_point_t` variable. E.g. `lv_point_t point = {10, 20};`. If the chart is scrolled the cursor will remain on the same place.

`lv_chart_get_point_pos_by_id(chart, series, id, &point_out)` tells the coordinate of a given point. It's useful to place the cursor to given point.

`lv_chart_set_cursor_point(chart, cursor, series, point_id)` sticks the cursor the point. If the point's position changes (new value or scrolling) the cursor will move with the point.  

## Events
- `LV_EVENT_VALUE_CHANGED` Sent when a new point is clicked perssed.  `lv_chart_get_pressed_point(chart)` returns the zero based index of the pressed point.
- `LV_EVENT_DRAW_PART_BEGIN` and `LV_EVENT_DRAW_PART_END` are sent for multiple parts. The fields of `lv_obj_draw_part_dsc_t` is set as the followings:
   - `LV_PART_ITEMS` (the series)
       - *Line chart* `clip_area`, `id` (index of the point), `value` (value of `id`th point), `p1`, `p2` (points of the line), `draw_area` (area of the point), `line_dsc`, `rect_dsc`, `sub_part_ptr` (pointer to the series), `part`
       - *Bar chart* `clip_area`, `id` (index of the point), `value` (value of `id`th point), `draw_area` (area of the point), `rect_dsc`, `sub_part_ptr` (pointer to the series), `part`
   - `LV_PART_TICKS` (major tick lines and label)  `clip_area`, `id` (axis), `value` (scaled value of the tick), `text` (`value` converted to decimal), `line_dsc`, `label_dsc`, `part`
   - `LV_PART_CURSOR` Te events are send for 3 times:
      - vertical line `clip_area`, `p1`, `p2` (points of the line), `line_dsc`, `part`
      - horizontal line `clip_area`, `p1`, `p2` (points of the line), `line_dsc`, `part`
      - point `clip_area`, `draw_area` (points of the line), `rect_dsc`, `part`
  - `LV_PART_MAIN` (the division lines)  `clip_area`, `id` (index of the line), `p1`, `p2` (points of the line), `line_dsc`, `part`
  
Learn more about [Events](/overview/event).

## Keys
No *Keys* are processed by the object type.

Learn more about [Keys](/overview/indev).

## Example

```eval_rst

.. include:: ../../../examples/widgets/chart/index.rst

```

## API

```eval_rst

.. doxygenfile:: lv_chart.h
  :project: lvgl

```
