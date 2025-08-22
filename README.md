 <h3 align="center">User Behavior Analysis</h3>

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#appendix">Appendix</a></li>
  </ol>
</details>

### Installation

How to Run the Code

1. Cleaning User Data, Cards Data, and Transactions Data with Python. Check the dataframe, missing data, and duplicate data.
2. Took 567,856 transaction from "transaction data" table as a sample.
3. Upload 3 tables to BigQuery: User Data, Cards Data, and Transactions Data.
4. Create "Master Table" to combine all tables.
<pre> WITH master_table AS (
    SELECT 
        t.id AS transaction_id,
        t.date,
        t.amount,
        t.use_chip,
        t.merchant_city,
        t.merchant_state,
        t.mcc,
        u.id AS user_id,
        u.current_age,
        u.retirement_age,
        u.gender,
        u.per_capita_income,
        u.yearly_income,
        u.total_debt,
        u.credit_score,
        u.num_credit_cards,
        c.card_brand,
        c.card_type,
        c.credit_limit,
        c.has_chip
    FROM `technical-test-mandiri-469703.technical_test.transactions_data` t
    LEFT JOIN `technical-test-mandiri-469703.technical_test.users_data` u
        ON t.client_id = u.id
    LEFT JOIN `technical-test-mandiri-469703.technical_test.cards_data` c
        ON t.card_id = c.id
)
SELECT *
FROM master_table </pre>
5. Make a query to get "user metrics cluster" table for analysis
   <pre> WITH master_table AS (
    SELECT 
        t.id AS transaction_id,
        t.amount,
        t.client_id,
        u.id AS user_id,
        u.current_age,
        u.gender,
        u.yearly_income,
        u.total_debt,
        u.num_credit_cards,
        c.credit_limit
    FROM `technical-test-mandiri-469703.technical_test.transactions_data` t
    LEFT JOIN `technical-test-mandiri-469703.technical_test.users_data` u
        ON t.client_id = u.id
    LEFT JOIN `technical-test-mandiri-469703.technical_test.cards_data` c
        ON t.card_id = c.id
     )
    SELECT
      user_id,
      COUNT(transaction_id) AS num_transactions,
      SUM(amount) AS total_spent,
      AVG(amount) AS avg_transaction,
      AVG(credit_limit) AS avg_credit_limit,
      SUM(amount) / NULLIF(AVG(credit_limit),0) AS utilization_rate,
      total_debt / NULLIF(yearly_income,0) AS debt_to_income_ratio,
      num_credit_cards,
      current_age,
      gender,
      yearly_income
     FROM master_table
     GROUP BY user_id, total_debt, yearly_income, num_credit_cards, current_age, gender, yearly_income</pre>

6. Make a query to get "cards channel" table for analysis
<pre> WITH master_table AS (
    SELECT 
        t.id AS transaction_id,
        t.date,
        t.amount,
        t.use_chip,
        t.merchant_city,
        t.merchant_state,
        t.mcc,
        t.card_id,
        u.id AS user_id,
        u.current_age,
        u.retirement_age,
        u.gender,
        u.per_capita_income,
        u.yearly_income,
        u.total_debt,
        u.credit_score,
        u.num_credit_cards,
        c.card_brand,
        c.card_type,
        c.credit_limit,
        c.has_chip
    FROM `technical-test-mandiri-469703.technical_test.transactions_data` t
    LEFT JOIN `technical-test-mandiri-469703.technical_test.users_data` u
        ON t.client_id = u.id
    LEFT JOIN `technical-test-mandiri-469703.technical_test.cards_data` c
        ON t.card_id = c.id
),
card_channel_metrics AS (
    SELECT
        card_type,
        card_brand,
        use_chip,
        COUNT(transaction_id) AS num_transactions,
        SUM(amount) AS total_spent
    FROM master_table
    GROUP BY card_type, card_brand, use_chip
)
  SELECT *
FROM card_channel_metrics
ORDER BY total_spent DESC
 </pre>
7. Make a query to get "location" table for analysis
<pre> WITH master_table AS (
    SELECT 
        t.id AS transaction_id,
        t.amount,
        t.client_id,
        t.merchant_city,
        t.merchant_state
    FROM `technical-test-mandiri-469703.technical_test.transactions_data` t
)
SELECT
    merchant_city,
    merchant_state,
    COUNT(transaction_id) AS num_transactions,
    SUM(amount) AS total_spent,
    COUNT(DISTINCT client_id) AS num_users
FROM master_table
GROUP BY merchant_city, merchant_state
ORDER BY total_spent DESC </pre>
8. After get "user metrics cluster", "cards channel, and "location" tables, create visualization on Looker.
9. Create deck presentation and analysis the user behavior.
10. Create a dashboard.


<p align="right">(<a href="#readme-top">back to top</a>)</p>


<!-- USAGE EXAMPLES -->
## Usage

This analysis aims to understand user behavior, focusing on transaction patterns (spending behavior) and credit risk. With proper user segmentation, the company can identify: 
High-value users, Potential risk groups, Spending patterns. 
<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- CONTACT -->
## Contact

Madeline Alexandra Angelica Barus - [Linkedln](http://linkedin.com/in/madelinexandra/) - madelinexandra30@gmail.com

<p align="right">(<a href="#readme-top">back to top</a>)</p>


<!-- APPENDIX -->
## Appendix


* [Python] (https://colab.research.google.com/drive/1o0MHxGmYGUGp-T8msCbi7xafb2JuBjo6?usp=sharing)
* [Deck] (https://www.canva.com/design/DAGwwlXML20/y7elF6l50DbjlJgBTOxE6Q/edit?utm_content=DAGwwlXML20&utm_campaign=designshare&utm_medium=link2&utm_source=sharebutton)
* [Dashboard] (https://lookerstudio.google.com/reporting/f683ec32-fc23-4e5e-a9a3-6aaffb80214f)
