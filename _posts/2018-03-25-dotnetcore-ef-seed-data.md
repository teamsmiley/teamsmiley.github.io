# EF Core  seed data

## DesignTimeDbContextFactory  ERROR 

DBContext 있는 폴더에 클래스를 만든다. 

```cs
public static class DBContextExtensions
    {
        public static void EnsureSeedDataForContext(this DBContext context)
        {
          
                //init seed data
                var costEstimator = new List<CostEstimator>()
                {
                    new CostEstimator()
                    {
                        Application = "maya",
                        Core = 12,
                        Email = "abcd@abcd.com",
                    },
                    new CostEstimator()
                    {
                        Application = "3dmax",
                        Core = 8,
                        Email = "12345",
                    },
                };
                context.CostEstimators.AddRange(costEstimator);
                context.SaveChanges();
            }

    }
```

Startup.cs에서 다음을 추가한다.

```cs
public void Configure(IApplicationBuilder app, IHostingEnvironment env,DBContext dbContext)
{
    await aaaContext.EnsureSeedDataForContext();
    app.UseMvc();
}
```
