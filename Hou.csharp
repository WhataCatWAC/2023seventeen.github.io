using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace MathPhysicsQuiz
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        public void ConfigureServices(IServiceCollection services)
        {
            services.AddDbContext<QuizContext>(options =>
                options.UseSqlite(Configuration.GetConnectionString("DefaultConnection")));
            
            services.AddControllers();
            services.AddCors(options =>
            {
                options.AddPolicy("AllowAll", builder =>
                {
                    builder.AllowAnyOrigin()
                           .AllowAnyMethod()
                           .AllowAnyHeader();
                });
            });
        }

        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            app.UseCors("AllowAll");
            app.UseStaticFiles();
            app.UseRouting();
            
            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
                endpoints.MapFallbackToFile("index.html");
            });
            
            // 初始化数据库
            using (var scope = app.ApplicationServices.CreateScope())
            {
                var dbContext = scope.ServiceProvider.GetRequiredService<QuizContext>();
                dbContext.Database.EnsureCreated();
            }
        }
    }

    public class QuizContext : DbContext
    {
        public QuizContext(DbContextOptions<QuizContext> options) : base(options) { }
        
        public DbSet<Question> Questions { get; set; }
    }

    public class Question
    {
        public int Id { get; set; }
        public string Type { get; set; } // "math" or "physics"
        public string Text { get; set; }
        public string OptionA { get; set; }
        public string OptionB { get; set; }
        public string OptionC { get; set; }
        public string OptionD { get; set; }
        public string CorrectAnswer { get; set; } // "A", "B", "C", or "D"
        public string Explanation { get; set; }
    }

    [ApiController]
    [Route("api/[controller]")]
    public class QuestionsController : ControllerBase
    {
        private readonly QuizContext _context;

        public QuestionsController(QuizContext context)
        {
            _context = context;
        }

        // 获取随机题目
        [HttpGet("random")]
        public async Task<ActionResult<Question>> GetRandomQuestion()
        {
            var count = await _context.Questions.CountAsync();
            if (count == 0) return NotFound("没有可用的题目");
            
            var random = new Random();
            var skip = random.Next(0, count);
            
            var question = await _context.Questions
                .OrderBy(q => q.Id)
                .Skip(skip)
                .FirstOrDefaultAsync();
                
            return question;
        }

        // 添加新题目
        [HttpPost]
        public async Task<ActionResult<Question>> AddQuestion([FromBody] Question question)
        {
            if (string.IsNullOrEmpty(question.Text) || 
                string.IsNullOrEmpty(question.OptionA) ||
                string.IsNullOrEmpty(question.OptionB) ||
                string.IsNullOrEmpty(question.OptionC) ||
                string.IsNullOrEmpty(question.OptionD) ||
                string.IsNullOrEmpty(question.CorrectAnswer) ||
                !new[] {"A", "B", "C", "D"}.Contains(question.CorrectAnswer))
            {
                return BadRequest("题目信息不完整或格式不正确");
            }

            _context.Questions.Add(question);
            await _context.SaveChangesAsync();
            
            return CreatedAtAction(nameof(GetRandomQuestion), new { id = question.Id }, question);
        }
    }
}
