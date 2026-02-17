## Summary:
•	Gold layer aggregation: computes product-level metrics from the Silver table, providing a consolidated view of all flash sale events.\
•	Behavioral analytics: tracks views, carts, and purchases per product/category, enabling conversion and cart-abandonment calculations.\
•	Key KPIs: calculates conversion percentage, cart abandonment percentage, demand pressure, and average order value for actionable insights.\
•	Trending analysis: generates recency-weighted trending scores to highlight high-demand products for marketing and inventory planning.

## Query:
```SQL
DROP TABLE IF EXISTS flash_sale_gold;

CREATE TABLE flash_sale_gold AS
SELECT
    ROW_NUMBER() OVER () AS id,

    product_id,
    category,

    -- total counts
    COUNT(*) AS total_events,
    COALESCE(SUM(quantity),0) AS total_units,
    COALESCE(SUM(revenue),0) AS total_revenue,

    -- behavior breakdown
    SUM(CASE WHEN event_type='view' THEN 1 ELSE 0 END) AS views,
    SUM(CASE WHEN event_type='cart' THEN 1 ELSE 0 END) AS carts,
    SUM(CASE WHEN event_type='purchase' THEN 1 ELSE 0 END) AS purchases,

    -- conversion rate % (purchase/view)
    COALESCE(
        ROUND(
            SUM(CASE WHEN event_type='purchase' THEN 1 ELSE 0 END)*100.0 /
            NULLIF(SUM(CASE WHEN event_type='view' THEN 1 ELSE 0 END),0)
        ,2)
    ,0) AS conversion_percentage,

    -- cart abandonment % (cart not purchased / cart)
    COALESCE(
        ROUND(
            (SUM(CASE WHEN event_type='cart' THEN 1 ELSE 0 END) -
             SUM(CASE WHEN event_type='purchase' THEN 1 ELSE 0 END)) *100.0 /
            NULLIF(SUM(CASE WHEN event_type='cart' THEN 1 ELSE 0 END),0)
        ,2)
    ,0) AS cart_abandon_percentage,

    -- demand pressure score
    COALESCE(
        ROUND(
            COUNT(*) * 1.0 / NULLIF(SUM(quantity),0)
        ,2)
    ,0) AS demand_pressure,

    -- avg order value per purchase
    COALESCE(
        ROUND(
            SUM(revenue) /
            NULLIF(SUM(CASE WHEN event_type='purchase' THEN 1 ELSE 0 END),0)
        ,2)
    ,0) AS avg_order_value,

    -- trending score (weighted by recency)
    SUM(
        CASE
            WHEN ts >= NOW() - INTERVAL 5 MINUTE THEN 5
            WHEN ts >= NOW() - INTERVAL 15 MINUTE THEN 3
            ELSE 1
        END
    ) AS trending_score

FROM flash_sale_silver
WHERE product_id IS NOT NULL
  AND category IS NOT NULL
  AND COALESCE(user_id,'') <> ''  -- remove any events without user ID
GROUP BY product_id, category;
```
