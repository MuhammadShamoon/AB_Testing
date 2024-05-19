# AB_Testing with SQL


### Project Overview
We are running an experiment at an item level, which means all users who visit will see the same page. However, the layout of different item pages may differ. We will calculate the order binary, view binary, average views and orders for the 30-day window after the test assignment for item_test_2.


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

The above SQL code gives the following analysis:
  
![image](https://github.com/MuhammadShamoon/AB_Testing/assets/52103515/e2e9cd1c-6699-4406-b480-83c5ba07d3f4)





### Recommendations

#### For Orders:

Using the above metrics, [ABBA](https://thumbtack.github.io/abba/demo/abba.html) (a statistical tool) gave the following results.

![image](https://github.com/MuhammadShamoon/AB_Testing/assets/52103515/72cba738-7681-4af3-a32b-41e2ee2fbe4b)

Based on the analysis of the success rates, the confidence intervals, and the p-value, it is clear that:

- There is no statistically significant difference between the control group and orders treatment (p-value of 0.88).
- The success rates are essentially the same, and the small negative improvement suggests no benefit from using the treatment version.
- The confidence intervals for both the control group and order treatment overlap, indicating no meaningful difference in performance.

The recommendation is to disapprove the variation. It does not provide any improvement over the control group, and the high p-value strongly suggests that any observed differences are due to chance rather than a real effect. Therefore, it is better to stick with the baseline or explore other variations that might lead to a significant improvement.

#### For Views: 

The following results are obtained using metrics from the analysis of views treatment.  

![image](https://github.com/MuhammadShamoon/AB_Testing/assets/52103515/08b7a6bd-4206-4b21-b6f8-71bd2b6c8d82)

From the analysis of the success rates, confidence intervals, and the p-value, the following points are clear:

- Views treatment has a slightly higher success rate (83%) compared to the control group (81%).
- The improvement is 2.6%, which is positive but relatively small.
- The p-value of 0.20 indicates that the observed improvement is not statistically significant.

Therefore, the recommendation is to disapprove the variation.
