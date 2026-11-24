-- Project: Cohort Analysis — Organic vs. Promotional User Retention
-- Database: PostgreSQL
-- Purpose: Data cleaning, cohort assignment,
-- monthly activity calculation and user aggregation.

WITH users_parsed AS (
    SELECT
        u.user_id,
        u.signup_datetime,
        u.promo_signup_flag,
        TO_DATE(
            LPAD(
                (REGEXP_MATCH(
                    REPLACE(
                        REPLACE(
                            REGEXP_REPLACE(TRIM(u.signup_datetime), ' [0-9]{1,2}:[0-9]{2}', ''),
                            '.', '/'
                        ),
                        '-', '/'
                    ),
                    '^([0-9]{1,2})/([0-9]{1,2})/([0-9]{2,4})$'
                ))[1],
                2, '0'
            )
            || '/' ||
            LPAD(
                (REGEXP_MATCH(
                    REPLACE(
                        REPLACE(
                            REGEXP_REPLACE(TRIM(u.signup_datetime), ' [0-9]{1,2}:[0-9]{2}', ''),
                            '.', '/'
                        ),
                        '-', '/'
                    ),
                    '^([0-9]{1,2})/([0-9]{1,2})/([0-9]{2,4})$'
                ))[2],
                2, '0'
            )
            || '/' ||
            CASE
                WHEN LENGTH(
                    (REGEXP_MATCH(
                        REPLACE(
                            REPLACE(
                                REGEXP_REPLACE(TRIM(u.signup_datetime), ' [0-9]{1,2}:[0-9]{2}', ''),
                                '.', '/'
                            ),
                            '-', '/'
                        ),
                        '^([0-9]{1,2})/([0-9]{1,2})/([0-9]{2,4})$'
                    ))[3]
                ) = 2
                THEN '20' ||
                    (REGEXP_MATCH(
                        REPLACE(
                            REPLACE(
                                REGEXP_REPLACE(TRIM(u.signup_datetime), ' [0-9]{1,2}:[0-9]{2}', ''),
                                '.', '/'
                            ),
                            '-', '/'
                        ),
                        '^([0-9]{1,2})/([0-9]{1,2})/([0-9]{2,4})$'
                    ))[3]
                ELSE
                    (REGEXP_MATCH(
                        REPLACE(
                            REPLACE(
                                REGEXP_REPLACE(TRIM(u.signup_datetime), ' [0-9]{1,2}:[0-9]{2}', ''),
                                '.', '/'
                            ),
                            '-', '/'
                        ),
                        '^([0-9]{1,2})/([0-9]{1,2})/([0-9]{2,4})$'
                    ))[3]
            END,
            'DD/MM/YYYY'
        )::timestamp AS signup_ts
    FROM project.cohort_users_raw u
),
events_parsed AS (
    SELECT
        e.user_id,
        e.event_datetime,
        e.event_type,
        e.revenue,
        TO_DATE(
            LPAD(
                (REGEXP_MATCH(
                    REPLACE(
                        REPLACE(
                            REGEXP_REPLACE(TRIM(e.event_datetime), ' [0-9]{1,2}:[0-9]{2}', ''),
                            '.', '/'
                        ),
                        '-', '/'
                    ),
                    '^([0-9]{1,2})/([0-9]{1,2})/([0-9]{2,4})$'
                ))[1],
                2, '0'
            )
            || '/' ||
            LPAD(
                (REGEXP_MATCH(
                    REPLACE(
                        REPLACE(
                            REGEXP_REPLACE(TRIM(e.event_datetime), ' [0-9]{1,2}:[0-9]{2}', ''),
                            '.', '/'
                        ),
                        '-', '/'
                    ),
                    '^([0-9]{1,2})/([0-9]{1,2})/([0-9]{2,4})$'
                ))[2],
                2, '0'
            )
            || '/' ||
            CASE
                WHEN LENGTH(
                    (REGEXP_MATCH(
                        REPLACE(
                            REPLACE(
                                REGEXP_REPLACE(TRIM(e.event_datetime), ' [0-9]{1,2}:[0-9]{2}', ''),
                                '.', '/'
                            ),
                            '-', '/'
                        ),
                        '^([0-9]{1,2})/([0-9]{1,2})/([0-9]{2,4})$'
                    ))[3]
                ) = 2
                THEN '20' ||
                    (REGEXP_MATCH(
                        REPLACE(
                            REPLACE(
                                REGEXP_REPLACE(TRIM(e.event_datetime), ' [0-9]{1,2}:[0-9]{2}', ''),
                                '.', '/'
                            ),
                            '-', '/'
                        ),
                        '^([0-9]{1,2})/([0-9]{1,2})/([0-9]{2,4})$'
                    ))[3]
                ELSE
                    (REGEXP_MATCH(
                        REPLACE(
                            REPLACE(
                                REGEXP_REPLACE(TRIM(e.event_datetime), ' [0-9]{1,2}:[0-9]{2}', ''),
                                '.', '/'
                            ),
                            '-', '/'
                        ),
                        '^([0-9]{1,2})/([0-9]{1,2})/([0-9]{2,4})$'
                    ))[3]
            END,
            'DD/MM/YYYY'
        )::timestamp AS event_ts
    FROM project.cohort_events_raw e
),
user_activity AS (
    SELECT
        u.user_id,
        u.promo_signup_flag,
        e.event_type,
        e.revenue,
        TO_CHAR(u.signup_ts, 'YYYY-MM') AS cohort_month,
        TO_CHAR(e.event_ts, 'YYYY-MM') AS activity_month,
        (
            EXTRACT(YEAR FROM e.event_ts) -
            EXTRACT(YEAR FROM u.signup_ts)
        ) * 12
        +
        (
            EXTRACT(MONTH FROM e.event_ts) -
            EXTRACT(MONTH FROM u.signup_ts)
        ) AS month_offset
    FROM users_parsed u
    JOIN events_parsed e
        ON u.user_id = e.user_id
    WHERE u.signup_ts IS NOT NULL
      AND e.event_ts IS NOT NULL
      AND e.event_type IS NOT NULL
      AND e.event_type <> 'test_event'
)
SELECT
    promo_signup_flag,
    cohort_month,
    month_offset,
    COUNT(DISTINCT user_id) AS users_total
FROM user_activity
WHERE activity_month >= '2025-01'
  AND activity_month <= '2025-06'
GROUP BY
    promo_signup_flag,
    cohort_month,
    month_offset
ORDER BY
    promo_signup_flag,
    cohort_month,
    month_offset;
