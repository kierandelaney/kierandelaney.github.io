---
layout: post
title: Counting Weekdays Since (or Until) a Date With PHP
categories: ["#php", "#snippet"]
---

For an internal project we have a series of entities which contain date properties and for various reasons we want to count how many working days have transpired since each date. In this case a working day is considered to be Monday through Friday, but this is defined in the function.

It took me long enough to put this together and test it that I thought it may be useful to someone else. Note, this makes no attempt at discounting public holidays (as we do not observe them in the context of "working days") but you could easily extend the below to skip public holidays generated from something like [yasumi](https://azuyalabs.github.io/yasumi/).

The function is also not particularly defensive (it will not test that it has actually been passed a date), but again that is not a concern in this instance.

```php
/**
 * test_weekdays - function to test whether or not a date is within x working days of today
 * @param  string $date_to_test - the date being tested
 * @return int                  - the number of days since or until the passed date
 */
function test_weekdays( string $date_to_test ) : int {

  $datetime_to_test = strtotime($date_to_test); // turn the input into a date regardless of incoming format
  $current_test_date = strtotime(date('Y-m-d', $datetime_to_test)); //output a date string (removing any time component)
  $today = strtotime(date('Y-m-d')); // turn the date back into a time now we've discarded any extra granularity

  $working_days = 0;
  $adjustment = "+1";                    // 1. assume it is in the past and we need to move forward

  if($current_test_date > $today){       // 2. unless it's future and we need to move backward
    $adjustment = "-1";
  }

  while ($current_test_date != $today) {
    if( in_array(date('N',$current_test_date), [1,2,3,4,5]) ){               // only count weekdays
      $working_days += ($adjustment*1);
    }
    $current_test_date = strtotime("{$adjustment} day", $current_test_date); // then traverse by one day
  }

  return $working_days;

}
```
