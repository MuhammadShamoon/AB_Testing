# AB_Testing with SQL


### Project Overview
We are running an experiment at an item level, which means all users who visit will see the same page. However, the layout of different item pages may differ. We will calculate the order binary, view binary and average views and orders for the 30-day window after the test assignment for item_test_2.


### Data Sources
The following datasets are meant to mimic an online eCommerce site, publically available at the [Mode Analytics](https://mode.com/).
- final_assignments.csv
- events.csv
- orders.csv

### Tools
- [Mode Analytics](https://mode.com/) is used to do SQL for this project.
- [ABBA](https://thumbtack.github.io/abba/demo/abba.html) is a statistical tool, which is used to compute the lifts in metrics and the p-values for the binary metrics (30-day order binary and 30-day view binary) using an interval 95% confidence. 

### SQL code 

```sql

SELECT
  test_assignment
  ,COUNT(item_id) AS items_control
  ,SUM(order_binary_30d) AS orders_treatment
  ,SUM(view_binary_30d) AS views_treatment
FROM
--Joining final_assignment, events and orders tables
  (SELECT 
    fa.item_id 
    ,test_assignment
--order_binary_30d tells that if an order is placed within 30 days after the test     
    ,MAX(CASE WHEN ((date(created_at) - date(test_start_date)) BETWEEN 0 AND 30)
                  THEN 1
                  ELSE NULL
                  END) AS order_binary_30d
    ,MAX(CASE WHEN ((date(event_time) - date(test_start_date)) BETWEEN 0 AND 30)
                  THEN 1 
                  ELSE NULL
                  END) AS view_binary_30d
  FROM 
    dsv1069.final_assignments fa
  
  LEFT JOIN 
    dsv1069.orders o 
    
    ON fa.item_id = o.item_id
  
  LEFT JOIN 

    (SELECT 
      MAX(CASE WHEN parameter_name = 'item_id' 
                  THEN CAST(parameter_value AS INT) 
                  ELSE NULL
                  END
          ) item_id
      ,event_time
      ,user_id
    FROM 
      dsv1069.events
    WHERE 
     event_name = 'view_item'
    GROUP BY 
      user_id
      ,event_time
    ) views
  
    ON fa.item_id = views.item_id
  
  WHERE
    test_number = 'item_test_2'
  
  GROUP BY 
    fa.item_id 
    ,test_assignment
  
  ) item_level
  
GROUP BY test_assignment

```

### Results

- The above SQL code gives the following analysis:
  
![image](https://github.com/MuhammadShamoon/AB_Testing/assets/52103515/e2e9cd1c-6699-4406-b480-83c5ba07d3f4)


- testing results 

### Recommendations


### Limitations
