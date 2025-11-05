# Analytics Dashboard Reference

> **Document Type**: Reference (SAMPLE)  
> **Audience**: Product Managers, Business Analysts, Administrators  
> **Last Updated**: November 5, 2024

---

## Overview

The Blog Engine Analytics Dashboard provides real-time insights into user behavior, content performance, and system health.

**Dashboard URL**: `https://app.blogengine.com/admin/analytics`

---

## Key Metrics

### User Metrics

#### Active Users
- **Daily Active Users (DAU)**: Unique users who logged in or created content in the last 24 hours
- **Weekly Active Users (WAU)**: Unique users who were active in the last 7 days
- **Monthly Active Users (MAU)**: Unique users who were active in the last 30 days

**Calculation**:
```sql
-- DAU
SELECT COUNT(DISTINCT user_id) 
FROM user_activity 
WHERE activity_date >= CURRENT_DATE - INTERVAL '1 day';

-- WAU
SELECT COUNT(DISTINCT user_id) 
FROM user_activity 
WHERE activity_date >= CURRENT_DATE - INTERVAL '7 days';
```

#### User Growth
- **New Registrations**: Users who signed up in the selected time period
- **Churn Rate**: Percentage of users who haven't been active in 30+ days
- **Retention Rate**: Percentage of users who return after first week

---

### Content Metrics

#### Post Performance
| Metric | Description | Calculation |
|--------|-------------|-------------|
| **Total Posts** | All published posts | `COUNT(*) FROM posts WHERE status = 'published'` |
| **Posts Today** | Posts published in last 24h | `COUNT(*) FROM posts WHERE published_at >= CURRENT_DATE` |
| **Average Posts/User** | Posts per active author | `Total Posts / Active Authors` |
| **Most Viewed Posts** | Top 10 by view count | `ORDER BY view_count DESC LIMIT 10` |

#### Engagement Metrics
- **Comments per Post**: Average number of comments per published post
- **Likes per Post**: Average number of likes per published post
- **Share Rate**: Percentage of posts that have been shared
- **Read Time**: Average time users spend reading posts

**Example Query**:
```sql
-- Top performing posts (last 30 days)
SELECT 
  p.id,
  p.title,
  COUNT(DISTINCT v.user_id) as unique_views,
  COUNT(DISTINCT c.id) as comment_count,
  COUNT(DISTINCT l.id) as like_count
FROM posts p
LEFT JOIN post_views v ON p.id = v.post_id
LEFT JOIN comments c ON p.id = c.post_id
LEFT JOIN post_likes l ON p.id = l.post_id
WHERE p.published_at >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY p.id, p.title
ORDER BY unique_views DESC
LIMIT 10;
```

---

### Traffic Metrics

#### Page Views
- **Total Page Views**: All page loads in selected period
- **Unique Visitors**: Distinct users/sessions viewing content
- **Bounce Rate**: Percentage of single-page sessions
- **Average Session Duration**: Mean time per user session

#### Traffic Sources
- **Direct**: Users who typed URL or used bookmark
- **Referral**: Traffic from other websites
- **Social**: Traffic from social media platforms
- **Search**: Traffic from search engines
- **Email**: Traffic from email campaigns

---

### Platform Metrics

#### Device Breakdown
```
Web Application: 65%
iOS Mobile App: 20%
Android Mobile App: 15%
```

#### Geographic Distribution
Top 5 countries by user count:
1. United States
2. United Kingdom
3. Canada
4. Germany
5. Australia

---

## Dashboard Sections

### 1. Overview Dashboard

**Widgets**:
- Total Users (current count)
- Daily Active Users (trend)
- Total Posts (current count)
- Total Comments (current count)
- System Health Status
- Recent Activity Feed

**Refresh Rate**: Real-time (WebSocket connection)

---

### 2. User Analytics

**Charts**:
- User Growth Over Time (line chart)
- User Acquisition Channels (pie chart)
- User Retention Cohort Analysis (heatmap)
- Active Users by Hour (bar chart)

**Filters**:
- Date Range: Last 7/30/90 days, Custom
- User Type: All, Free, Premium
- Registration Date: Range selector

**Export**: CSV, PDF

---

### 3. Content Analytics

**Charts**:
- Post Publication Frequency (line chart)
- Top Authors by Posts (bar chart)
- Top Categories by Views (pie chart)
- Engagement Rate Trends (line chart)

**Tables**:
- Top 100 Posts by Views
- Trending Posts (last 24 hours)
- Underperforming Posts (low engagement)

**Actions**:
- Feature Post
- Pin to Homepage
- Archive Post
- Delete Post

---

### 4. Engagement Analytics

**Metrics**:
- Comment Rate: Comments / Post Views
- Like Rate: Likes / Post Views
- Share Rate: Shares / Post Views
- Average Read Time per Post

**Breakdown By**:
- Post Category
- Author
- Time of Day
- Day of Week

---

### 5. Traffic Analytics

**Real-time**:
- Active Users Right Now (live counter)
- Current Page Views (live map)
- Active Sessions by Location

**Historical**:
- Page Views Over Time
- Traffic Sources Breakdown
- Top Landing Pages
- Top Exit Pages

**Integration**: Google Analytics embed

---

### 6. Revenue Analytics (Future)

**Metrics**:
- Monthly Recurring Revenue (MRR)
- Average Revenue Per User (ARPU)
- Customer Lifetime Value (CLV)
- Conversion Rate (Free → Paid)

---

## API Endpoints

### Get Dashboard Metrics

```
GET /api/admin/analytics/metrics
```

**Parameters**:
- `start_date` (string): ISO 8601 date
- `end_date` (string): ISO 8601 date
- `metrics` (array): List of metric names

**Example Request**:
```bash
curl -X GET "https://api.blogengine.com/admin/analytics/metrics?start_date=2024-11-01&end_date=November 5, 2024&metrics=dau,post_count" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Example Response**:
```json
{
  "start_date": "2024-11-01",
  "end_date": "November 5, 2024",
  "metrics": {
    "dau": {
      "2024-11-01": 1250,
      "2024-11-02": 1320,
      "2024-11-03": 1180,
      "2024-11-04": 1420,
      "November 5, 2024": 1390
    },
    "post_count": {
      "2024-11-01": 45,
      "2024-11-02": 52,
      "2024-11-03": 38,
      "2024-11-04": 61,
      "November 5, 2024": 48
    }
  }
}
```

---

### Get Top Posts

```
GET /api/admin/analytics/top-posts
```

**Parameters**:
- `limit` (integer): Number of posts (default: 10, max: 100)
- `period` (string): Time period (`day`, `week`, `month`)
- `metric` (string): Ranking metric (`views`, `comments`, `likes`)

---

### Get User Cohort Data

```
GET /api/admin/analytics/cohorts
```

**Parameters**:
- `cohort_type` (string): `weekly` or `monthly`
- `start_date` (string): Cohort start date

**Response**: Retention matrix by cohort

---

## Permissions

### Required Roles
- **Admin**: Full access to all analytics
- **Editor**: Access to content analytics only
- **Author**: Access to own content analytics only

### Audit Logging
All analytics data exports are logged:
```json
{
  "action": "analytics_export",
  "user_id": 123,
  "report_type": "user_growth",
  "date_range": "2024-11-01 to November 5, 2024",
  "timestamp": "November 5, 2024T14:30:00Z"
}
```

---

## Data Retention

- **Real-time data**: 24 hours
- **Daily aggregates**: 1 year
- **Weekly aggregates**: 3 years
- **Monthly aggregates**: Indefinite

---

## Export Options

### CSV Export
- Click "Export" button on any dashboard
- Select date range
- Choose metrics to include
- File delivered via email (large exports)

### Scheduled Reports
Configure automated weekly/monthly reports:
1. Go to Settings → Reports
2. Select metrics and frequency
3. Add email recipients
4. Save

---

## Related

- [Admin Panel Guide](../how-to/admin/manage-content.md)
- [User Management](user-management.md)
- [API Reference](../api/endpoints.md)
