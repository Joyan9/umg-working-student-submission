# UMG Product Analyst Challenge

### Important Links
* Link to Script: https://colab.research.google.com/drive/1uxSJVZykEZwvFODMjSZs4hjQ92zzJWJ4?usp=sharing
* Link to Dashboard: https://lookerstudio.google.com/reporting/cc1a7ae8-f7e8-410e-b0b8-79f609781ef3

### How I Would Approach the Problem

1. Identify individual tasks within the problem statement and understand the business impact of each outcome.
2. Choose the right tools, considering their efficiency gains. Given mixed file formats and large data volumes that cannot be processed locally, I would use a distributed computing engine such as Apache Spark.
3. Address each task separately:

#### 1. TOP 50 Monthly Tracks by Country (Central Europe, Last 3 Months)

* Determine the ranking criteria (number of plays, average duration, etc.).
* Tables required:

  * Use the `plays` table for activity data.
  * Join with `users` on `user_id` via broadcast join to get `country`.
  * Since `tracks` lacks a displayable name, use `track_id` from `plays` as the identifier.

#### 2. TOP 10 Artists by Genre (Last 30 Days)

* Assume "Top 10" by highest number of plays.
* Tables required:

  * Filter `plays` to the last 30 days before joining.
  * Join with `tracks` to group by `genre`.

#### 3. A/B Testing for Listening Duration

* **Hypothesis:** Personalized playlist recommendations increase average listening duration compared to generic recommendations.

  **Control Group (A):**
  Users see the standard Home feed with generic, popularity-based recommendations (no age-group section).

  **Variant Group (B):**
  Users see an "Age-Group Favorites" section at the top of their Home feed. It is populated with the top 8 tracks most played by users in the same `age_group` over the past 14 days.

  **Metrics to Track**

  * **Primary Metric:** Average session duration (total `play_duration_seconds` per session, averaged across all sessions in each cohort).
  * **Secondary Metrics:**

    * Skip rate (number of plays with < 30s duration ÷ total plays).
    * Songs per session (total tracks started ÷ total sessions).
    * Return rate (% of users who start a new session within 24 hours of their last session).
    * Completion rate (full-track plays ÷ total plays).

  **Key Steps**

  1. **Randomization:** 50/50 split by hashing `user_id`.
  2. **Instrumentation:**

     * Log `age_picks_impressed` on Home load.
     * Log `age_picks_click` when a user taps a track in the age-group section.
     * Log all `play_start`/`play_end` events with `user_id`, `session_id`, and variant flag.
  3. **Analysis Window:** Run for approximately two weeks or until 10,000 sessions per cohort.
  4. **Statistical Test:** Two-sample t-test (or non-parametric equivalent) on average session duration, with p < 0.05 as the threshold for significance.

### How I Would Run the Solution

* I created a script in a [Google Colab notebook](https://colab.research.google.com/drive/1uxSJVZykEZwvFODMjSZs4hjQ92zzJWJ4?usp=sharing) to generate the required views and store the output as CSV (which can be changed to any other format for easier visualization).
* The script can be scheduled to run automatically via a cron job or scheduler.

### How I Would Verify the Solution

* Compare results against a source of truth if available; otherwise, manually validate a subset of results against expected outputs.

### Pros and Cons of My Solution

**Pros**

* Spark can handle large datasets efficiently.
* Caching and broadcast joins optimize query performance.

**Cons**

* Requires setup and maintenance of a Spark cluster.
* Not well suited for real-time dashboards, since Spark cluster startup can take several minutes.

### How I Would Optimize the Solution

* **In terms of Query Optimization:-** Filter data early; leverage date partitioning if available.
* **For Storage Optimization:-** Store views in Parquet for better compression and partition by relevant columns (e.g., country for the Top 50 tracks view).

### Other Considerations

* Ensure that PII data is secure both at rest and in transit.
