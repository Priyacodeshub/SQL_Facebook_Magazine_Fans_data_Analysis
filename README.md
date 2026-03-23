Facebook Magazine Fan Data Analysis — SQL Business Intelligence Project
Tools: PostgreSQL | SQL | Tableau (Dashboard)
Techniques: JOINs, CTEs, Window Functions, Aggregations, Date-Time Logic, Subqueries
Dataset: 7 tables | 150 countries | 46 cities | 41 languages | 10,000+ rows

Business Problem
A French-language magazine runs a Facebook page with a global audience spanning 150 countries. The marketing team needed answers to three strategic questions:

Where are our fans, and where is our brand strongest relative to population?
Who are our fans — demographics, language, and buying power?
When should we publish content for maximum engagement?

Additionally, leadership wanted to evaluate the feasibility of launching a US English-language edition by estimating the potential buying power of English-speaking fans in the United States.

Database Schema
TableDescriptionKey ColumnsPopStatsReference data for 150 countriesCountryCode, CountryName, Population, AverageIncomeFansPerCountryFan counts by country over timeDate, CountryCode, NumberOfFansFansPerCityFan counts across 46 citiesDate, City, CountryCode, NumberOfFansFansPerGenderAgeFan demographicsDate, Gender, AgeGroup, NumberOfFansFansPerLanguageFan language preferences (41 languages)Date, Language, CountryCode, NumberOfFansGlobalPageDaily page performance (45 rows/day)Date, CountryCode, NewLikes, DailyPostsReachPostInsightsPost-level engagement metricsCreatedTime, EngagedFans, PostClicks, Reach

Key Findings
1. Methodology Correction — Daily Average Reach
Discovered a critical aggregation error in the original reach calculation:

Incorrect approach: AVG(DailyPostsReach) directly → 1,862,816 (this averaged per-country rows, not global daily totals)
Corrected approach: SUM per date first, then AVG of daily totals → 83,826,721

The GlobalPage table contains ~45 rows per day (one per country). Averaging without first summing to daily totals underestimated reach by 45x. This correction demonstrates why understanding data granularity matters before writing any aggregation.
sqlWITH globaldaily AS (
    SELECT date, SUM(dailypostsreach) AS globaldailyreach
    FROM globalpage
    GROUP BY date
)
SELECT ROUND(AVG(globaldailyreach), 2) AS dailypostreach
FROM globaldaily;
-- Result: 83,826,721.36
2. Raw Fans ≠ Market Penetration
Top countries by raw fan count and by penetration ratio tell completely different stories:
Metric#1 CountryValueRaw FansIvory Coast112,160 fansPenetration RatioReunion2.41%
Ivory Coast leads in absolute fans but has only ~0.46% penetration (population: 24M). Reunion has just 20,885 fans but a 2.41% penetration ratio (population: 866K). Penetration ratio is the more meaningful metric for identifying where the brand has genuine market presence.
sqlSELECT p.countryname,
       ROUND((f.numberoffans * 100.0) / p.population, 2) AS penetration_ratio,
       f.numberoffans, p.population
FROM fanspercountry f
JOIN popstats p ON f.countrycode = p.countrycode
WHERE date = (SELECT MAX(date) FROM fanspercountry)
ORDER BY penetration_ratio DESC
LIMIT 10;
3. Audience Demographics

Gender: 56.46% female, 43.44% male
Age: Largest group is 25–34 (35.73%), followed by 18–24 (21.3%)
Profile: Young, female-skewing audience — aligns with lifestyle magazine positioning

4. Language and US Buying Power

English-speaking fans: 1,347,752 (only 5.08% of total fan base)
Estimated US buying power: $5,464,504.63
The brand's core audience is non-English, aligning with French-African positioning
US English edition is viable but represents a niche segment

sqlSELECT ROUND(
    SUM(f.NumberOfFans * p.AverageIncome * 0.0001), 2
) AS PotentialBuyingPower
FROM FansPerLanguage f
JOIN PopStats p ON f.CountryCode = p.CountryCode
WHERE f.Language = 'en'
  AND LOWER(p.CountryName) = 'united states';
-- Result: $5,464,504.63
5. Engagement Patterns

Best time to post: 05:00–08:59 drives 33.96% of total engagement
Day-of-week caveat: Posts are published almost entirely on the 10th of each month, so day-of-week results reflect calendar coincidence rather than genuine engagement patterns


SQL Techniques Demonstrated

JOINs — Combining fan data with population stats for penetration analysis
CTEs (Common Table Expressions) — Multi-step aggregations for corrected reach calculation
Window Functions — DENSE_RANK, ROW_NUMBER for ranking countries and segments
Subqueries — Filtering to most recent date using SELECT MAX(date)
Aggregations — SUM, AVG, COUNT, ROUND with GROUP BY for summary statistics
Date-Time Logic — Extracting day-of-week and time-of-day from timestamps
Data Quality Awareness — Identifying and correcting aggregation methodology errors


All 16 Business Questions
#SectionQuestionKey ResultQ1Basic SummaryUnique countries150Q2Basic SummaryUnique cities46Q3Basic SummaryUnique languages41Q4Basic SummaryDaily average reach83,826,721 (corrected)Q5Basic SummaryDaily average engagement (NewLikes)8,942.56Q6Location AnalysisTop 10 countries by fansIvory Coast #1 (112,160)Q7Location AnalysisTop 10 countries by penetration ratioReunion #1 (2.41%)Q8Location AnalysisBottom 10 cities (growth potential)Bejaia, Algeria (2,301 fans)Q9Fan AnalysisFan split by age group25–34 leads (35.73%)Q10Fan AnalysisFan split by genderFemale 56.46%, Male 43.44%Q11Language AnalysisEnglish-speaking fans1,347,752Q12Language AnalysisEnglish fans as % of total5.08%Q13Language AnalysisUS English buying power$5,464,504.63Q14EngagementEngagement by day of weekSaturday #1 (19.73%)Q15EngagementEngagement by time of day05:00–08:59 #1 (33.96%)Q16AdvancedMonth-to-month engagement change(window function challenge)

Project Structure
├── README.md
├── SQL Queries/                                              -- All SQL query files
├── Dhandapani_Rathina Priya_Business_queries_022026.pdf      -- 16 business questions with queries and results
└── Dhandapani_Rathina Priya_3_presentation_022026.pdf        -- Visual presentation of findings

About Me
Rathina Priya Dhandapani — Data Analyst | Associate Banker at JPMorgan Chase
B.Sc. Mathematics | SQL, Tableau, Python

LinkedIn - www.linkedin.com/in/priya-dhandapani-067417342
