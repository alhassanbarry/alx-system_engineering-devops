My first Postmortem

Outage Duration:

Start Time: 2023-10-15 14:30 UTC
End Time: 2023-10-15 16:15 UTC
Duration: 1 hour 45 minutes

Impact:
-The web application became unresponsive for 85% of users, resulting in failed login attempts, slow page loads, and timeouts.

-API response times increased to 15-20 seconds (normal: 200ms).

-Users were unable to complete transactions, leading to a 20% drop in revenue during the outage.

Root Cause:
The database connection pool was exhausted due to a misconfigured connection timeout setting, causing all available connections to remain open and unavailable for new requests.
Timeline
14:30 UTC: Issue detected when monitoring alerts flagged a spike in database connection errors and increased API response times.

14:35 UTC: Engineering team notified via PagerDuty. Initial assumption was a database server overload.

14:40 UTC: Database server CPU and memory usage checked; both were within normal limits.

14:50 UTC: Application logs reviewed, revealing a high number of "connection timeout" errors.

15:00 UTC: Misleading path: Investigated network latency between application servers and the database, but no anomalies found.

15:15 UTC: Incident escalated to the Database and DevOps teams for further investigation.

15:30 UTC: Root cause identified: Database connection pool exhaustion due to a misconfigured timeout setting.

15:45 UTC: Configuration updated to reduce connection timeout from 10 minutes to 1 minute.
16:00 UTC: Restarted application servers to release stuck connections.

16:15 UTC: Full service restored, confirmed by monitoring tools and user reports.
Root Cause and Resolution
Root Cause:
The database connection pool was exhausted because the connection timeout setting was too high (10 minutes). This caused connections to remain open even after queries were completed, preventing new connections from being established. As traffic increased, the pool was fully consumed, leading to connection errors and application unresponsiveness.

Resolution:

1.Reduced the connection timeout setting from 10 minutes to 1 minute to ensure connections are released faster.

2.Restarted the application servers to clear all stuck connections and reset the connection pool.

3.Verified the fix by monitoring database connection metrics and API response times, which returned to normal levels.

Corrective and Preventative Measures
Improvements/Fixes:

1.Optimize database connection pool settings to better handle traffic spikes.

2.Implement better monitoring for connection pool usage and database performance.

3.Conduct regular load testing to identify bottlenecks before they impact production.

TODO List:

1.Patch application servers to use dynamic connection pool sizing based on traffic.

2.Add monitoring for database connection pool usage (e.g., alerts when usage exceeds 80%).

3.Update documentation to include best practices for database connection management.
4.Schedule a postmortem review with the engineering team to discuss lessons learned.
