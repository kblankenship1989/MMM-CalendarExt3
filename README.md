# MMM-CalendarExt3
MagicMirror module for calendar view.




## Screenshot




## Concept

My previous module `MMM-CalendarExt2` was always notorious with its difficulty to use. I need more easy and light one. So I rewrite from scratch. 


## Features
### What's different with `CX2`
- Only focusing to how it shows; Parsing is delegated original MagicMirror module `calendar`. (It means, `calendar` module is REQURIRED to use this module.)
- Only `week` and `month` views. I found that people are rare interested in other views on `CX2`. So I omit other views.
- Respect to original MM's hide/show mechanism. Now you can hide/show this module easily with other scheduler or control module. (By the way, Look this module also. - [MMM-Scenes](https://github.com/MMRIZE/MMM-Scenes))
- No dependency to the 3rd party modules (e.g. momentJS or Luxon, ...). Built with pure JS and CSS only.

### Main Features
- `week` view or `month` view
- locale-aware calendar
- customizing events : filtering, sorting, transforming
- multi instance available. You don't need copy and rename module. Just add one more configuration in your `config.js`


## Install
```sh
cd ~/MagicMirror/modules
git clone https://github.com/MMRIZE/MMM-CalendarExt3
```

## Config
Anyway even this simplest will work.
```js
{
  module: "MMM-CalendarExt3",
  position: "bottom_center",
},

```

More conventional;
```js
{
  module: "MMM-CalendarExt3",
  position: "bottom_center",
  title: "",
  config: {
    mode: "month",
    instanceId: "basicCalendar",
    locale: 'de-DE',
    maxEventLines: 5,
    firstDayOfWeek: 1,
    calendarSet: ['us_holiday', 'abfall', 'mytest'],
    ...
  }
},
```

You need setup default `calendar` configuration also.
```js
/* default/calendar module configuration */
{
  module: "calendar",
  position: "top_left",
  config: {
    broadcastPastEvents: true, // <= IMPORTANT to see past events
    calendars: [
      {
        url: "webcal://www.calendarlabs.com/ical-calendar/ics/76/US_Holidays.ics",
        name: "us_holiday", // <= RECOMMENDED to assign name
        color: "red" // <= RECOMMENDED to assign color
      },
      ...

```

### Config details
All the properties are omittable, and if omitted, a default value will be applied.

|**property**|**default**|**description**|
|---|---|---|
|`mode`| 'week' | Calendar view type. You can choose between 'week' and 'month'|
|`weekIndex`| -1 | Which week starts in a view. `-1` is the previous week of current focusing moment. `0` is the focusing week of the moment. `1` will be the next week, and so on.<br>Ignored on `mode:'month'`|
|`weeksInView` | 3 | How many weeks from the index. <br> `weekIndex:-1`, `weeksInView:3` means 3 weeks view from the last week. <br> Ignored on `mode:'month'`|
|`instanceId` | (auto-generated) | When you want more than 1 instance of this module, each instance would need this value to distinguish each others. If you doesn't assign this property, `identifier` of the module instance will be assigned automatically, but not recommended to use it. (Hard to guess the implicited value.)|
|`firstDayOfWeek`| 1 | Monday is the first day of the week according to the international standard ISO 8601, but in the US, Canada, Japan and some cultures, it's counted as the second day of the week. If you want to start the week from Monday, set this property to `1`. If you want Sunday, set `0`. <br> Sunday:0, Monday:1, Tuesday:2, ..., Saturday:6 |
|`minimalDaysOfNewYear` | 4 | ISO 8601 also says **each week's year is the Gregorian year in which the Thursday falls**. The first week of the year, hence, always contains 4 January. However, the US (Yes, it is.) system is different with standards. In US, **containing 1 January** defines the first week. In that case set this value to `1`. And under the some other culture, you might need to modify this. |
|`locale` | (`language` of MM config) | `de` or `ko-KR` or `ja-Jpan-JP-u-ca-japanese-hc-h12`. It defines how to handle and display your date-time values by the locale. When omitted, the default `language` config value of MM. |
|`calendarSet` | [] | When you want to display only selected calendars, fulfill this array with the targeted calendar name(of the default `calendar` module). <br>e.g) `calendarSet: ['us_holiday', 'office'],`<br> `[]` or `null` will allow all the calendars. |
|`maxEventLines` | 5 | How much events will be displayed in 1 day cell. The overflowed events will be hidden. |
|`fontSize` | `18px` | Default font size of this module. |
|`eventHeight` | `22px` | The height of each event. |
|`cellDateOptions` | {month: 'short', <br>day: 'numeric'} | The format of day cell date. It varies by the `locale` and this option. <br>`locale:'en-US'`, the default displaying will be something like `Jun 1` or `1`. <br> See [options](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat/DateTimeFormat#parameters) | 
|`eventTimeOptions` | {timeStyle: 'short'} | The format of event time. It varies by the `locale` and this option. <br> `locale:'en-US'`, the default displaying will be something like `3:45 pm`.<br> See [options](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat/DateTimeFormat#parameters) | 
|`headerWeekDayOptions`|{weekday: 'long'} | The format of weekday header. It varies by the `locale` and this option. <br> `locale:'en-US'`, the default displaying will be something like `Tuesday`.<br> See [options](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat/DateTimeFormat#parameters) |
|`headerTitleOptions`|{month: 'long'} | The format of module header of the month view. It varies by the `locale` and this option. <br> `locale:'en-US'`, the default displaying will be something like `December`.<br> See [options](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat/DateTimeFormat#parameters) |
|`eventFitler`| callback function | See the `Filtering` part.|
|`eventSorter`| callback function | See the `Sorting` part.|
|`eventTransformer`| callback function | See the `Transforming` part.|

## Notification
### Incoming Notifications
#### `CX3_MOVE_CALENDAR`, payload: {instanceId, step}
Move calendar to another moment. 
- `instanceId` : If you have more than 1 instance of this module, you can specify the instance to manipulate. If omitted or `null`, all instances would obey this notification order. 
- `step` : How many leaps of the current view. In `mode:'week'` 1 step will be a week. In `mode:'month'` 1 step will be a month. Negative value be allowed.

#### `CX3_SET_DATE`, payload: {instanceId, date}
Set the date of specific view. 
- `instanceId` : See the above
- `date` : Specific date to move. e.g) In `mode:'month'`, `date: "2022-12-25"` would display calendar view of `2022 Decemeber`. Be aware of `weekIndex` of `mode:'week'`. The view range will be calculated with `weekIndex`, `weeksInView` and this value.

#### `CALENDAR_EVENTS`
Any module which can emit this notification could the source of this module. Generally, default `calendar` module would be.

### Outgoing Notification
Nothing yet.  (Does it need?)

## Styling with CSS
You can handle almost all of visual stuffs with CSS. See the `module.css` and override your needs into your `custom.css`.
- `CX3`, `CX3_{instanceId}`, `mode_week` or `mode_month` : The root selector. Each instance of this module will have `CX3_{instanceId}` as another root selector. With this CSS selector, you can assign individual look to each instance.
```css
.CX3 {
  /* you CAN modify these values; but SHOULD NOT to remove */
  --celllinecolor: #333;
  --cellbgcolor: rgba(0, 0, 0, 0.2);
  --cellheaderheight: 25px;
  --cellfooterheight: 2px;
  --defaultcolor: #FFF;
  --eventheight: calc(var(--fontsize) + 4px);
  --totalheight: calc(var(--eventheight) * var(--maxeventlines));
  font-size: var(--fontsize);
  color: var(--defaultcolor);
  line-height: calc(var(--eventheight))
}
```
Most common used values would be defined in `.CX3` selector as variables. 

`--fontsize`, `--maxeventlines`, `--eventheight` would be imported from configuration for your setup convenience.

- `.cell` : Every day cell has this selector. Each cell could have these class name together by its condition.
  - `.today`, `.thisMonth`, `.thisYear`
  - `.year_2022`, `.month_12`, `.date_25`, `.weekday_0`
- `.cellHeader`, `.cellFooter` : Parts of day cell. `.cellHeader` would have `.cw`(count of week) and `.cellDate` as children.
  - `.cellHeader .cellDate` : Displaying date of the cell. The date would have many parts of date/hour information from `cellDateOptions` of config.

- `.event` : Every event has this selector. Each event could have these class name together by its condition.
  - `.continueFromPreviousWeek`, `.continueToNextWeek`
  - `.calendar_{calendarName}`, `{class}` : Orginal `calendar`
  - `.passed`, `.future`, `.current`, 
  - `.multiday`, `.singleday`, `.fullday`

And `event` also has `dataSet` (`data-*`) as its attributes. (e.g. data-title="...", data-start-date="...") You can use these attributes also.

- `.weekGrid`, `.weekGridRow` : Defition of calendar grid. You can adjust the grid itself. (e.g. Shrink the width of weekends cells)

## Handling Events
Each event object has this structure.
```json
{
  "title": "Leeds United - Chelsea",
  "startDate": 1650193200000,
  "endDate": 1650199500000,
  "fullDayEvent": false,
  "class": "PUBLIC",
  "location": false,
  "geo": false,
  "description": "...",
  "today": false,
  "symbol": ["calendar-alt"],
  "calendarName": "tottenham",
  "color": "gold",
  "calendarSeq": 1, // This would be the order from `calendarSet` of configuration
  "isPassed": true,
  "isCurrent": false,
  "isFuture": false,
  "isFullday": false,
  "isMultiday": false
}
```
You can use these values to handle events.

### Filtering
You can filter each event by its condition.
```js
eventFilter: (ev) => {
  if (ev.title.isFullday) return false
  return true
}
```
This example shows how you can filter out some event which is 'fullday' event.

### Sorting
You can sort each event by its condition. However, this module arrange events masonly with density. So displaying would not fit with your sorting intention. Anyway, try it if you need.
```js
eventSorter: (a, b) => {
  return a.calendarSeq - b.calendarSeq
}
```
This example tries to sort event by calendar order in `calendarSet`.

### Transforming
You can manipulate or change the properties of the event.
```js
eventTransformer: (ev) => {
  if (ev.title.search('John') > -1) ev.color = 'blue'
  return ev
}
```
This example shows how you can transform the color of events when the event title has specific text.


## Not the bug, but...
- Weirdly, the default `calendar` module doesn't emit the exact starting time of `multidays-fullday event including current moment`. Always it would be the current  .... Writing ...
