# EF Core  seed data

## DesignTimeDbContextFactory  ERROR 

DBContext 있는 폴더에 클래스를 만든다. 

```cs
public static class DBContextExtensions
    {
        public static async Task EnsureSeedDataForContext(this DBContext context)
        {
            if (!await context.Boards.AnyAsync()) // 데이터가 잇는지 확인 없으면 다음 진행
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
    }
```

Startup.cs에서 다음을 추가한다.

```cs
public void Configure(IApplicationBuilder app, IHostingEnvironment env,DBContext dbContext)
{
    dbContext.EnsureSeedDataForContext();
    app.UseMvc();
}
```
