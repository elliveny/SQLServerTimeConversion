# SQLServerTimeConversion
A C# project which generates SQL Server functions which convert UTC to Local times and back again. To jump straight into the code it generates see [Example.sql](https://github.com/elliveny/SQLServerTimeConversion/blob/master/Example.sql).

A number of SQL Server projects which I've worked on needed a method of converting a UTC time to a user's local time when given that user's timezone. To that end I wanted a function to provide me with the conversion, like this:

    DECLARE @usersTimezone VARCHAR(32)='Europe/London'
    DECLARE @utcDT DATETIME=GetUTCDate()
    DECLARE @userDT DATETIME=[dbo].[funcUTCtoLocal](@utcDT, @usersTimezone)

I needed the @usersTimezone value to be any IANA zone supported by NodaTime - see the list here: https://nodatime.org/TimeZones

Be aware that my use case means I only need a 'current' window, allowing the conversion of times within the range of about +/- 5 years from now.  This means that the method I've used probably isn't suitable for you if you need a very wide period of time, due to the way it generates code for each timezone interval in a given date range.

I also needed the reverse conversion:

    DECLARE @usersTimezone VARCHAR(32)='Europe/London'
    DECLARE @userDT DATETIME=GetDate()
    DECLARE @utcDT DATETIME=[dbo].[funcLocaltoUTC](@userDT, @usersTimezone)

To this end I prepared this .NET Core project, which generates the two SQL Server functions from the information in NodaTime TZDB data.

See example output from the project in the file [Example.sql](https://github.com/elliveny/SQLServerTimeConversion/blob/master/Example.sql).

The generation is controlled by the statics at the top of [Program.cs](https://github.com/elliveny/SQLServerTimeConversion/blob/master/Program.cs):

    static IEnumerable<NodaTime.DateTimeZone> timeZones = DateTimeZoneProviders.Tzdb.GetAllZones();
    static Instant fromInstant = Instant.FromUtc(2015, 1, 1, 0, 0);
    static Instant toInstant = Instant.FromUtc(2025, 12, 1, 0, 0);

If you need a subset of the timeZones, or if you require a different range of supported dates then you can make changes to these variables to achieve that.

