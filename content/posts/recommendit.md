+++
date = '2025-05-20T12:29:17+02:00'
title = 'Recommendit'
+++


**TECH STACK: C#, SQL Server, EF Core, MongoDB, Python(BERT Model), Angular**


Recommednit is a project I had been working on and off for around a year now.

As a __TV Show enjoyer__,a fan of many classic shows like [The Wire](https://www.youtube.com/watch?v=P3i36ybA8Ms&ab_channel=HBO), [The Sopranos](https://www.youtube.com/watch?v=YsBipoG22Nw&ab_channel=jerksto), [Sucession](https://www.youtube.com/watch?v=7GGajk7cwYw&t=1s&ab_channel=RabbitWhisperer) I wanted to make a small project that would suggest shows based on a users' top 3. Eventually, through some research I found [cosine similarity](https://datastax.medium.com/how-to-implement-cosine-similarity-in-python-505e8ec1d823), a formula that calculates measure of similarity between two non-zero vectors in an n-dimensional space. 
This was the base of my application. But the question was how do I quantify shows to numbers.

This is when LLMs came to aid. BERT ML Model (initially I used Word2Vec, a slower, less efficient one) would convert the descriptions of every show into a vector.

All that was left was for the user to type 3 of his favorite shows, after which the program would do the calculation based on those show descriptions (which were converted to vectors).

Query Perfromance was a problem here. With a project like this, I had a lot of opportunity to practice with a large amount of seed data. This is more closer to projects in real life. With Recommendit, I borrowed a free copy of a database from [TheTVDB](https://www.thetvdb.com/), a popular database with over 200 000 shows. The first operation that required tuning was searching:

- User can query and select up to 3 shows from those. 
  This is where performance was a tiny problem, but easily solved after I learned a bit about indices in SQL Server. The fast solution here was to cluster over the show name to allow for seeks over the large database of shows.

- User should receive 10 different show recommendations based on the 3 initial ones that they picked.

The .NET code with the initial queries:


    
    List<int> userShowIds = new List<int> { id1, id2, id3 };
    List<double[]?> userShowsVectors = userShowIds
        .Select(id => ShowService.GetVectorById(_context, id))
        .Where(vector => vector != null)
        .ToList();

    List<ShowInfo> allShows = ShowService.GetShowInfos(_context,userShowIds); //exclude shows that are already in the 1st list!
   
    double[] averageVector = VectorEngine.CalculateAverageVector(userShowsVectors);
    List<int> recommendedShowIds = await VectorEngine.GetSimilarities(allShows,averageVector,8);
       return Ok(recommendedShowIds);
    

I learned a lot here, because initially I wrote queries inside a for loop, made round trips to the database causing n+1 query issues.
After delving deep into SQL indexes, specifically [SQL Server: Indexing For Performance](https://app.pluralsight.com/library/courses/sqlserver-indexing-for-performance/table-of-contents), taught by one of the people who was in the SQL server development team, with a lot of great information about index internals I was ready to revisit the project.

But even after solving all of this, the query performance was still slow, taking around 25 seconds on my i7 16GB laptop, and literally timing out on my Linux server.
The problem was that the columns were too large to be indexed, spanning over 8000 characters.

Finally I just switched to using a noSQL database, such as MongoDB for the querying and the calculation part, and this is what solved the performance issues. This is due to MongoDb's fast in-memory document retrieval and sequential scans without relationships and relational constraints.

The query now lasts around 6 or 7 seconds, last time I checked. See for yourself - https://recommendit.xyz

Overall a great project where I've learned a tons of different things.