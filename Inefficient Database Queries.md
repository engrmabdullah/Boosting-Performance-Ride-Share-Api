<p><strong>🚀</strong> <strong>Finding available drivers within a specific radius</strong></p>

<hr>
<p><strong>❌</strong> <strong>Problem: Inefficient Database Queries</strong></p>
<p>For a ride-share app, finding available drivers within a specific radius can be slow if the database queries aren’t optimized.</p>
<p><strong>🚫</strong> <strong>Inefficient Code:</strong></p>
<pre><code>public async Task&lt;List&lt;Driver&gt;&gt; GetAvailableDrivers(double userLat, double userLong, double radius) {

var allDrivers = await _dbContext.Drivers.ToListAsync(); // Fetching all drivers

return allDrivers.Where(driver =&gt; CalculateDistance(userLat, userLong, driver.Lat, driver.Long) &lt;= radius).ToList();
}

private double CalculateDistance(double lat1, double lon1, double lat2, double lon2) {

var R = 6371; // Earth radius in km

var dLat = (lat2 - lat1) * (Math.PI / 180);

var dLon = (lon2 - lon1) * (Math.PI / 180);

var a = Math.Sin(dLat / 2) * Math.Sin(dLat / 2) +

Math.Cos(lat1 * (Math.PI / 180)) * Math.Cos(lat2 * (Math.PI / 180)) *

Math.Sin(dLon / 2) * Math.Sin(dLon / 2);

var c = 2 * Math.Atan2(Math.Sqrt(a), Math.Sqrt(1 - a));

return R * c;
}
</code></pre>
<hr>
<p><strong>✅</strong> <strong>Best Practice: Optimize the Query for the Database</strong></p>
<p>Shift the heavy computation to the database by using SQL spatial functions like ST_Distance.</p>
<p><strong>✅</strong> <strong>Optimized Code:</strong></p>
<pre><code>public async Task&lt;List&lt;Driver&gt;&gt; GetAvailableDrivers(double userLat, double userLong, double radius){

var userLocation = new Point(userLat, userLong) { SRID = 4326 }; // GeoPoint for the user
return await _dbContext.Drivers
    .Where(driver =&gt; driver.Location.IsWithinDistance(userLocation, radius))
    .ToListAsync();
    }
</code></pre>
<hr>
<p><strong>Changes Made:</strong></p>
<p>1️⃣ <strong>Spatial Data Types:</strong> Ensure the Location field in your Driver entity is stored as a Geometry or Geography type in the database.</p>
<p><strong>Driver Entity Example:</strong></p>
<pre><code>public class Driver {
public int Id { get; set; }    
public string Name { get; set; }   
public Point Location { get; set; } // Driver's geo-location  
public bool IsAvailable { get; set; }   
}
</code></pre>
<p>2️⃣ <strong>Database Indexing:</strong> Add a <strong>spatial index</strong> on the Location field for faster query execution.</p>
<p><strong>Add Spatial Index:</strong></p>
<pre><code>protected override void Up(MigrationBuilder migrationBuilder) {
migrationBuilder.Sql("CREATE SPATIAL INDEX IX_Location ON Drivers (Location)");
}
</code></pre>
<p>3️⃣ <strong>Leverage SQL Spatial Functions:</strong> Use built-in functions like ST_Distance for distance calculation.</p>
<hr>
<p><strong>🚀</strong> <strong>Performance Impact:</strong></p>
<ul>
<li><strong>Before Optimization:</strong> 3 seconds per query, consuming 80% of memory for 50,000 drivers.</li>
<li><strong>After Optimization:</strong> 100ms per query with minimal memory usage for the same dataset.</li>
</ul>
<hr>
<p><strong>🔧</strong> <strong>Additional Tips for Ride-Share Systems</strong></p>
<p>1️⃣ <strong>Use Caching for Frequent Data:</strong><br>
Example using Redis:</p>
<pre><code>var cacheKey = $"driver:{driverId}:status";
var status = await _cache.GetStringAsync(cacheKey);
if (status == null)  {
    status = await GetDriverStatusFromDb(driverId);
    await _cache.SetStringAsync(cacheKey, status, new DistributedCacheEntryOptions
    {
    AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5)
    });
}
</code></pre>
<p>2️⃣ <strong>Minimize Middleware Overhead:</strong><br>
Use a minimal middleware pipeline:</p>
<pre><code>    app.UseRouting();
    app.UseAuthentication();
    app.UseAuthorization();
    app.UseEndpoints(endpoints =&gt; endpoints.MapControllers());
</code></pre>
<p>3️⃣ <strong>Batch Driver Location Updates:</strong><br>
Update driver locations in batches instead of individual updates:</p>
<pre><code>var driverUpdates = new List&lt;Driver&gt; {
    new Driver { Id = 1, Location = new Point(37.7749, -122.4194) { SRID = 4326 } },
    new Driver { Id = 2, Location = new Point(34.0522, -118.2437) { SRID = 4326 } }
    };

_dbContext.Drivers.UpdateRange(driverUpdates);
await _dbContext.SaveChangesAsync();
</code></pre>

