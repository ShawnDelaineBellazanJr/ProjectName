
# Comprehensive Technical Documentation: Building AI-Powered Applications with .NET Aspire and Semantic Kernel

## Table of Contents
1. [Introduction](#introduction)
2. [Architecture Overview](#architecture-overview)
3. [Setting Up the Development Environment](#setting-up-the-development-environment)
4. [Project Structure & Clean Architecture](#project-structure--clean-architecture)
5. [Semantic Kernel Integration](#semantic-kernel-integration)
6. [Implementing AI Planning](#implementing-ai-planning)
7. [Building API Services with .NET Aspire](#building-api-services-with-net-aspire)
8. [Data Access and Repository Pattern](#data-access-and-repository-pattern)
9. [Prompt Engineering & Management](#prompt-engineering--management)
10. [Testing Strategy](#testing-strategy)
11. [Deployment and DevOps](#deployment-and-devops)
12. [Performance Optimization](#performance-optimization)
13. [Security Best Practices](#security-best-practices)
14. [Monitoring and Observability](#monitoring-and-observability)
15. [Conclusion and Next Steps](#conclusion-and-next-steps)

## Introduction

This documentation provides a comprehensive guide for building AI-powered applications using .NET Aspire and Semantic Kernel. In this guide, you'll learn how to create cloud-native applications that leverage large language models (LLMs) with a focus on scalability, maintainability, and developer productivity.

### Target Audience
- .NET developers transitioning to AI development
- Software architects designing AI-integrated systems
- Development teams implementing Semantic Kernel applications

### Key Technologies
- .NET 9 with .NET Aspire
- Semantic Kernel
- Entity Framework Core
- gRPC for high-performance services
- SQL Server / PostgreSQL for persistent storage
- Docker and Kubernetes for containerization
- Azure OpenAI Service / OpenAI API

## Architecture Overview

### Cloud-Native AI Architecture

The architecture follows a microservices approach with .NET Aspire as the orchestration framework. The system comprises the following components:

```
┌─────────────────────────────────────────────────────────────┐
│                    Client Applications                      │
└───────────────────────────────┬─────────────────────────────┘
                                │
┌───────────────────────────────▼─────────────────────────────┐
│                          API Gateway                        │
└───────┬─────────────────────┬────────────────────────┬──────┘
        │                     │                        │
┌───────▼─────────┐   ┌───────▼─────────┐      ┌───────▼─────────┐
│  Auth Service   │   │   AI Service    │      │  Data Service   │
└───────┬─────────┘   └───────┬─────────┘      └───────┬─────────┘
        │                     │                        │
        │             ┌───────▼─────────┐              │
        │             │ Semantic Kernel │              │
        │             │    Runtime      │              │
        │             └───────┬─────────┘              │
        │                     │                        │
┌───────▼─────────┐   ┌───────▼─────────┐      ┌───────▼─────────┐
│  User Database  │   │  Prompt Store   │      │ Application DB  │
└─────────────────┘   └─────────────────┘      └─────────────────┘
```

### Component Responsibilities

1. **API Gateway**: Routes client requests to appropriate microservices
2. **Auth Service**: Handles authentication and authorization
3. **AI Service**: Core service integrating Semantic Kernel for AI capabilities
4. **Data Service**: Manages application data and business logic
5. **Semantic Kernel Runtime**: Orchestrates AI model interactions and planning
6. **Prompt Store**: Database for storing and managing prompts
7. **Application DB**: Stores application-specific data

## Setting Up the Development Environment

### Prerequisites

1. Install the .NET 9 SDK
   ```bash
   dotnet --list-sdks
   # Ensure .NET 9 SDK is installed
   ```

2. Install the .NET Aspire workload
   ```bash
   dotnet workload install aspire
   ```

3. Install Docker Desktop for containerization
   ```bash
   # Docker Desktop download link: https://www.docker.com/products/docker-desktop
   ```

4. Set up your preferred IDE (Visual Studio 2022+ or JetBrains Rider recommended)

### Creating a New .NET Aspire Solution

```bash
dotnet new aspire-starter --name AIAspireSolution --output AIAspireSolution
cd AIAspireSolution
```

### Adding Required NuGet Packages

```bash
# For Semantic Kernel
dotnet add package Microsoft.SemanticKernel
dotnet add package Microsoft.SemanticKernel.Abstractions

# For Database Access
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Design

# For API Development
dotnet add package Microsoft.AspNetCore.Grpc.JsonTranscoding
dotnet add package Microsoft.AspNetCore.Grpc.Swagger

# For Testing
dotnet add package Microsoft.NET.Test.Sdk
dotnet add package xunit
dotnet add package Moq
```

## Project Structure & Clean Architecture

### Solution Structure

Follow this structure for a clean architecture approach:

```
AIAspireSolution/
├── src/
│   ├── AIAspireSolution.AppHost/               # .NET Aspire host
│   ├── AIAspireSolution.ServiceDefaults/       # Shared service configurations
│   ├── AIAspireSolution.ApiGateway/            # API Gateway project
│   ├── AIAspireSolution.AiService/             # AI Service with Semantic Kernel
│   │   ├── Controllers/                        # API endpoints
│   │   ├── Services/                           # Business logic 
│   │   ├── Planning/                           # SK planning implementations
│   │   └── Plugins/                            # Semantic Kernel plugins
│   ├── AIAspireSolution.DataService/           # Data access service
│   │   ├── Controllers/                        # API endpoints
│   │   ├── Services/                           # Business logic
│   │   ├── Data/                               # EF Core context and config
│   │   └── Repositories/                        # Repository implementations
│   └── AIAspireSolution.Domain/                # Shared domain entities
│       ├── Models/                             # Domain models
│       ├── Interfaces/                         # Interface definitions
│       └── DTOs/                               # Data transfer objects
├── tests/
│   ├── AIAspireSolution.AiService.Tests/       # AI Service unit tests
│   ├── AIAspireSolution.DataService.Tests/     # Data Service unit tests
│   └── AIAspireSolution.IntegrationTests/      # Integration tests
└── tools/                                     # Build and deployment tools
```

### Domain Layer Implementation

Create clean, well-defined domain models:

```csharp
// AIAspireSolution.Domain/Models/Prompt.cs
namespace AIAspireSolution.Domain.Models;

public class Prompt
{
    public Guid Id { get; set; } = Guid.NewGuid();
    public string Name { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
    public string Template { get; set; } = string.Empty;
    public string ModelId { get; set; } = string.Empty;
    public decimal Temperature { get; set; } = 0.7m;
    public int MaxTokens { get; set; } = 1000;
    public Dictionary<string, string> DefaultParameters { get; set; } = new();
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime? UpdatedAt { get; set; }
}
```

```csharp
// AIAspireSolution.Domain/Models/Completion.cs
namespace AIAspireSolution.Domain.Models;

public class Completion
{
    public Guid Id { get; set; } = Guid.NewGuid();
    public Guid PromptId { get; set; }
    public string Input { get; set; } = string.Empty;
    public string Output { get; set; } = string.Empty;
    public Dictionary<string, string> Parameters { get; set; } = new();
    public int TokensUsed { get; set; }
    public decimal Cost { get; set; }
    public TimeSpan LatencyMs { get; set; }
    public string ModelId { get; set; } = string.Empty;
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public virtual Prompt? Prompt { get; set; }
}
```

### Interface Definitions

Define clear interfaces for service abstractions:

```csharp
// AIAspireSolution.Domain/Interfaces/IPromptService.cs
namespace AIAspireSolution.Domain.Interfaces;

public interface IPromptService
{
    Task<Prompt?> GetPromptAsync(Guid id);
    Task<Prompt?> GetPromptByNameAsync(string name);
    Task<IEnumerable<Prompt>> GetAllPromptsAsync();
    Task<Prompt> CreatePromptAsync(Prompt prompt);
    Task<Prompt> UpdatePromptAsync(Prompt prompt);
    Task<bool> DeletePromptAsync(Guid id);
    Task<string> RenderPromptAsync(Guid id, Dictionary<string, string> parameters);
}
```

## Semantic Kernel Integration

### Registering Semantic Kernel Services

Configure Semantic Kernel in the DI container:

```csharp
// AIAspireSolution.AiService/Program.cs
using Microsoft.SemanticKernel;

var builder = WebApplication.CreateBuilder(args);

// Add service defaults & Aspire components
builder.AddServiceDefaults();

// Configure Semantic Kernel
builder.Services.AddSingleton(sp =>
{
    var config = sp.GetRequiredService<IConfiguration>();
    
    // Create kernel builder
    var kernelBuilder = Kernel.CreateBuilder();
    
    // Add OpenAI or Azure OpenAI services
    if (config["AI:Provider"] == "AzureOpenAI")
    {
        kernelBuilder.AddAzureOpenAIChatCompletion(
            deploymentName: config["AI:AzureOpenAI:DeploymentName"]!,
            endpoint: config["AI:AzureOpenAI:Endpoint"]!,
            apiKey: config["AI:AzureOpenAI:ApiKey"]!
        );
    }
    else
    {
        kernelBuilder.AddOpenAIChatCompletion(
            modelId: config["AI:OpenAI:ModelId"] ?? "gpt-4",
            apiKey: config["AI:OpenAI:ApiKey"]!
        );
    }
    
    // Build and return the kernel
    return kernelBuilder.Build();
});

// Add application services
builder.Services.AddScoped<IPromptService, PromptService>();
builder.Services.AddScoped<ICompletionService, CompletionService>();
builder.Services.AddScoped<IAIPlanner, SemanticKernelPlanner>();

// Add controllers & other services
builder.Services.AddControllers();
builder.Services.AddGrpc().AddJsonTranscoding();
builder.Services.AddGrpcSwagger();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure middleware
app.UseExceptionHandler();
app.UseSwagger();
app.UseSwaggerUI();
app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.MapGrpcService<AiGrpcService>();

app.Run();
```

### Creating Semantic Kernel Plugins

Custom plugins extend Semantic Kernel functionality:

```csharp
// AIAspireSolution.AiService/Plugins/TimePlugin.cs
using System.ComponentModel;
using Microsoft.SemanticKernel;

namespace AIAspireSolution.AiService.Plugins;

public class TimePlugin
{
    [KernelFunction, Description("Get the current date and time.")]
    public string GetCurrentTime()
    {
        return DateTime.UtcNow.ToString("yyyy-MM-dd HH:mm:ss UTC");
    }
    
    [KernelFunction, Description("Get the current date.")]
    public string GetCurrentDate()
    {
        return DateTime.UtcNow.ToString("yyyy-MM-dd");
    }
    
    [KernelFunction, Description("Calculate the time difference between two dates.")]
    public string GetTimeDifference(
        [Description("Start date in yyyy-MM-dd format")] string startDate,
        [Description("End date in yyyy-MM-dd format")] string endDate)
    {
        if (!DateTime.TryParse(startDate, out var start) || !DateTime.TryParse(endDate, out var end))
        {
            return "Invalid date format. Please use yyyy-MM-dd format.";
        }
        
        var difference = end - start;
        return $"The difference is {difference.Days} days.";
    }
}
```

### Registering Plugins with Semantic Kernel

```csharp
// AIAspireSolution.AiService/Services/KernelService.cs
using Microsoft.SemanticKernel;

namespace AIAspireSolution.AiService.Services;

public class KernelService
{
    private readonly Kernel _kernel;
    
    public KernelService(Kernel kernel)
    {
        _kernel = kernel;
        
        // Register plugins
        _kernel.ImportPluginFromObject(new TimePlugin(), "TimePlugin");
        _kernel.ImportPluginFromObject(new DataPlugin(), "DataPlugin");
        
        // You can also register plugins from prompts
        RegisterPromptPlugins();
    }
    
    private void RegisterPromptPlugins()
    {
        // Register plugins from prompt templates stored in database or files
        // Implementation depends on your prompt storage strategy
    }
    
    public Kernel GetKernel() => _kernel;
}
```

## Implementing AI Planning

### Creating a Planner Service

Semantic Kernel planners help orchestrate complex AI-driven workflows:

```csharp
// AIAspireSolution.AiService/Planning/SemanticKernelPlanner.cs
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Planning;
using AIAspireSolution.Domain.Interfaces;

namespace AIAspireSolution.AiService.Planning;

public class SemanticKernelPlanner : IAIPlanner
{
    private readonly Kernel _kernel;
    private readonly ILogger<SemanticKernelPlanner> _logger;
    
    public SemanticKernelPlanner(Kernel kernel, ILogger<SemanticKernelPlanner> logger)
    {
        _kernel = kernel;
        _logger = logger;
    }
    
    public async Task<PlanningResponse> CreatePlanAsync(string goal)
    {
        try
        {
            _logger.LogInformation("Creating plan for goal: {Goal}", goal);
            
            // Create a planner
            var planner = new FunctionCallingStepwisePlanner(_kernel);
            
            // Generate an execution plan
            var plan = planner.CreatePlan(goal);
            
            // Execute the plan
            var result = await plan.InvokeAsync(_kernel);
            
            return new PlanningResponse
            {
                WasSuccessful = true,
                Result = result.GetValue<string>(),
                Plan = plan.ToString()
            };
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error creating plan for goal: {Goal}", goal);
            
            return new PlanningResponse
            {
                WasSuccessful = false,
                ErrorMessage = ex.Message
            };
        }
    }
    
    public async Task<PlanningResponse> ExecuteManualPlanAsync(string[] steps, Dictionary<string, string> variables)
    {
        try
        {
            _logger.LogInformation("Executing manual plan with {StepCount} steps", steps.Length);
            
            var kernelArguments = new KernelArguments();
            foreach (var kvp in variables)
            {
                kernelArguments[kvp.Key] = kvp.Value;
            }
            
            string result = string.Empty;
            
            // Execute each step sequentially
            foreach (var step in steps)
            {
                // Create a function from the step
                var function = _kernel.CreateFunctionFromPrompt(step);
                
                // Execute the function
                var stepResult = await _kernel.InvokeAsync(function, kernelArguments);
                
                // Store result for the next step
                result = stepResult.GetValue<string>() ?? string.Empty;
                kernelArguments["previousResult"] = result;
            }
            
            return new PlanningResponse
            {
                WasSuccessful = true,
                Result = result
            };
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error executing manual plan");
            
            return new PlanningResponse
            {
                WasSuccessful = false,
                ErrorMessage = ex.Message
            };
        }
    }
}

public class PlanningResponse
{
    public bool WasSuccessful { get; set; }
    public string Result { get; set; } = string.Empty;
    public string Plan { get; set; } = string.Empty;
    public string ErrorMessage { get; set; } = string.Empty;
}
```

### Implementing a Multi-Step Planning Workflow

```csharp
// AIAspireSolution.AiService/Services/WorkflowService.cs
using AIAspireSolution.Domain.Interfaces;
using AIAspireSolution.Domain.Models;

namespace AIAspireSolution.AiService.Services;

public class WorkflowService
{
    private readonly IAIPlanner _planner;
    private readonly IPromptService _promptService;
    private readonly ILogger<WorkflowService> _logger;
    
    public WorkflowService(
        IAIPlanner planner,
        IPromptService promptService,
        ILogger<WorkflowService> logger)
    {
        _planner = planner;
        _promptService = promptService;
        _logger = logger;
    }
    
    public async Task<WorkflowResult> ExecuteContentGenerationWorkflowAsync(ContentGenerationRequest request)
    {
        _logger.LogInformation("Starting content generation workflow for topic: {Topic}", request.Topic);
        
        try
        {
            // Step 1: Research and outline generation
            var researchPrompt = await _promptService.GetPromptByNameAsync("ContentResearch");
            var researchPlan = new string[]
            {
                await _promptService.RenderPromptAsync(researchPrompt.Id, new Dictionary<string, string>
                {
                    { "topic", request.Topic },
                    { "audience", request.TargetAudience },
                    { "style", request.ContentStyle }
                })
            };
            
            var researchResult = await _planner.ExecuteManualPlanAsync(researchPlan, new Dictionary<string, string>());
            if (!researchResult.WasSuccessful)
            {
                throw new Exception($"Research step failed: {researchResult.ErrorMessage}");
            }
            
            // Step 2: Content creation based on research
            var contentPrompt = await _promptService.GetPromptByNameAsync("ContentCreation");
            var contentPlan = new string[]
            {
                await _promptService.RenderPromptAsync(contentPrompt.Id, new Dictionary<string, string>
                {
                    { "topic", request.Topic },
                    { "research", researchResult.Result },
                    { "tone", request.Tone },
                    { "wordCount", request.WordCount.ToString() }
                })
            };
            
            var contentResult = await _planner.ExecuteManualPlanAsync(contentPlan, new Dictionary<string, string>());
            if (!contentResult.WasSuccessful)
            {
                throw new Exception($"Content creation step failed: {contentResult.ErrorMessage}");
            }
            
            // Step 3: Review and refinement
            var reviewPrompt = await _promptService.GetPromptByNameAsync("ContentReview");
            var reviewPlan = new string[]
            {
                await _promptService.RenderPromptAsync(reviewPrompt.Id, new Dictionary<string, string>
                {
                    { "content", contentResult.Result },
                    { "guidelines", request.AdditionalGuidelines }
                })
            };
            
            var reviewResult = await _planner.ExecuteManualPlanAsync(reviewPlan, new Dictionary<string, string>());
            
            return new WorkflowResult
            {
                Success = true,
                Content = reviewResult.WasSuccessful ? reviewResult.Result : contentResult.Result,
                Metadata = new Dictionary<string, string>
                {
                    { "researchData", researchResult.Result },
                    { "reviewFeedback", reviewResult.WasSuccessful ? reviewResult.Result : "Review step was skipped" }
                }
            };
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error in content generation workflow for topic: {Topic}", request.Topic);
            
            return new WorkflowResult
            {
                Success = false,
                ErrorMessage = ex.Message
            };
        }
    }
}

public class ContentGenerationRequest
{
    public string Topic { get; set; } = string.Empty;
    public string TargetAudience { get; set; } = string.Empty;
    public string ContentStyle { get; set; } = string.Empty;
    public string Tone { get; set; } = string.Empty;
    public int WordCount { get; set; } = 500;
    public string AdditionalGuidelines { get; set; } = string.Empty;
}

public class WorkflowResult
{
    public bool Success { get; set; }
    public string Content { get; set; } = string.Empty;
    public string ErrorMessage { get; set; } = string.Empty;
    public Dictionary<string, string> Metadata { get; set; } = new();
}
```

## Building API Services with .NET Aspire

### Creating gRPC Services

gRPC services offer high-performance APIs:

```protobuf
// AIAspireSolution.AiService/Protos/ai_service.proto
syntax = "proto3";

option csharp_namespace = "AIAspireSolution.AiService.Protos";

package ai;

import "google/api/annotations.proto";

service AiService {
  // Generate text content
  rpc GenerateContent(GenerateContentRequest) returns (GenerateContentResponse) {
    option (google.api.http) = {
      post: "/v1/ai/generate"
      body: "*"
    };
  }
  
  // Create a plan for a complex task
  rpc CreatePlan(CreatePlanRequest) returns (CreatePlanResponse) {
    option (google.api.http) = {
      post: "/v1/ai/plan"
      body: "*"
    };
  }
}

message GenerateContentRequest {
  string prompt = 1;
  map<string, string> parameters = 2;
  string model_id = 3;
  float temperature = 4;
  int32 max_tokens = 5;
}

message GenerateContentResponse {
  string content = 1;
  int32 tokens_used = 2;
  string model_used = 3;
  float latency_seconds = 4;
}

message CreatePlanRequest {
  string goal = 1;
  repeated string available_tools = 2;
}

message CreatePlanResponse {
  bool success = 1;
  string plan = 2;
  string error_message = 3;
}
```

### Implementing the gRPC Service

```csharp
// AIAspireSolution.AiService/Services/AiGrpcService.cs
using AIAspireSolution.AiService.Protos;
using Microsoft.SemanticKernel;
using Grpc.Core;

namespace AIAspireSolution.AiService.Services;

public class AiGrpcService : Protos.AiService.AiServiceBase
{
    private readonly Kernel _kernel;
    private readonly IAIPlanner _planner;
    private readonly ILogger<AiGrpcService> _logger;
    
    public AiGrpcService(Kernel kernel, IAIPlanner planner, ILogger<AiGrpcService> logger)
    {
        _kernel = kernel;
        _planner = planner;
        _logger = logger;
    }
    
    public override async Task<GenerateContentResponse> GenerateContent(
        GenerateContentRequest request, ServerCallContext context)
    {
        _logger.LogInformation("Received generate content request with prompt: {Prompt}", 
            request.Prompt.Length > 50 ? request.Prompt[..50] + "..." : request.Prompt);
        
        var startTime = DateTime.UtcNow;
        
        try
        {
            // Create kernel arguments from parameters
            var arguments = new KernelArguments();
            foreach (var param in request.Parameters)
            {
                arguments[param.Key] = param.Value;
            }
            
            // Create function from prompt
            var function = _kernel.CreateFunctionFromPrompt(
                request.Prompt,
                new PromptExecutionSettings
                {
                    ModelId = string.IsNullOrEmpty(request.ModelId) ? null : request.ModelId,
                    Temperature = request.Temperature,
                    MaxTokens = request.MaxTokens > 0 ? request.MaxTokens : null
                });
            
            // Execute function
            var result = await _kernel.InvokeAsync(function, arguments);
            var content = result.GetValue<string>() ?? string.Empty;
            
            // Calculate latency
            var latency = (DateTime.UtcNow - startTime).TotalSeconds;
            
            // Create response (token calculation would require model-specific calculation)
            return new GenerateContentResponse
            {
                Content = content,
                TokensUsed = EstimateTokenCount(request.Prompt) + EstimateTokenCount(content),
                ModelUsed = request.ModelId,
                LatencySeconds = (float)latency
            };
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error generating content");
            throw new RpcException(new Status(StatusCode.Internal, ex.Message));
        }
    }
    
    public override async Task<CreatePlanResponse> CreatePlan(
        CreatePlanRequest request, ServerCallContext context)
    {
        _logger.LogInformation("Received create plan request with goal: {Goal}", request.Goal);
        
        try
        {
            var planResult = await _planner.CreatePlanAsync(request.Goal);
            
            return new CreatePlanResponse
            {
                Success = planResult.WasSuccessful,
                Plan = planResult.Plan,
                ErrorMessage = planResult.ErrorMessage
            };
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error creating plan");
            throw new RpcException(new Status(StatusCode.Internal, ex.Message));
        }
    }
    
    private int EstimateTokenCount(string text)
    {
        // This is a very rough estimate: ~4 characters per token for English text
        return text.Length / 4;
    }
}
```

### REST API Controllers

In addition to gRPC, implement REST controllers for broader compatibility:

```csharp
// AIAspireSolution.AiService/Controllers/PromptController.cs
using Microsoft.AspNetCore.Mvc;
using AIAspireSolution.Domain.Models;
using AIAspireSolution.Domain.Interfaces;

namespace AIAspireSolution.AiService.Controllers;

[ApiController]
[Route("api/[controller]")]
public class PromptController : ControllerBase
{
    private readonly IPromptService _promptService;
    private readonly ILogger<PromptController> _logger;
    
    public PromptController(IPromptService promptService, ILogger<PromptController> logger)
    {
        _promptService = promptService;
        _logger = logger;
    }
    
    [HttpGet]
    [ProducesResponseType(typeof(IEnumerable<Prompt>), StatusCodes.Status200OK)]
    public async Task<IActionResult> GetAllPrompts()
    {
        var prompts = await _promptService.GetAllPromptsAsync();
        return Ok(prompts);
    }
    
    [HttpGet("{id}")]
    [ProducesResponseType(typeof(Prompt), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> GetPromptById(Guid id)
    {
        var prompt = await _promptService.GetPromptAsync(id);
        if (prompt == null)
        {
            return NotFound();
        }
        return Ok(prompt);
    }
    
    [HttpGet("name/{name}")]
    [ProducesResponseType(typeof(Prompt), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> GetPromptByName(string name)
    {
        var prompt = await _promptService.GetPromptByNameAsync(name);
        if (prompt == null)
        {
            return NotFound();
        }
        return Ok(prompt);
    }
    
    [HttpPost]
    [ProducesResponseType(typeof(Prompt), StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<IActionResult> CreatePrompt([FromBody] Prompt prompt)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }
        
        var createdPrompt = await _promptService.CreatePromptAsync(prompt);
        return CreatedAtAction(nameof(GetPromptById), new { id = createdPrompt.Id }, createdPrompt);
    }
    
    [HttpPut("{id}")]
    [ProducesResponseType(typeof(Prompt), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<IActionResult> UpdatePrompt(Guid id, [FromBody] Prompt prompt)
    {
        if (id != prompt.Id)
        {
            return BadRequest("ID mismatch");
        }
        
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }
        
        var existingPrompt = await _promptService.GetPromptAsync(id);
        if (existingPrompt == null)
        {
            return NotFound();
        }
        
        var updatedPrompt = await _promptService.UpdatePromptAsync(prompt);
        return Ok(updatedPrompt);
    }
    
    [HttpDelete("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> DeletePrompt(Guid id)
    {
        var prompt = await _promptService.GetPromptAsync(id);
        if (prompt == null) ...
    }

    // AIAspireSolution.DataService/Data/AppDbContext.cs
using Microsoft.EntityFrameworkCore;
using AIAspireSolution.Domain.Models;

namespace AIAspireSolution.DataService.Data;

/// <summary>
/// Application database context providing access to all entity collections
/// and configuration for Entity Framework Core
/// </summary>
public class AppDbContext : DbContext
{
    /// <summary>
    /// Collection of AI prompts stored in the database
    /// </summary>
    public DbSet<Prompt> Prompts { get; set; } = null!;
    
    /// <summary>
    /// Collection of AI completions/responses with usage tracking
    /// </summary>
    public DbSet<Completion> Completions { get; set; } = null!;
    
    /// <summary>
    /// Collection of user feedback on AI-generated content
    /// </summary>
    public DbSet<Feedback> Feedback { get; set; } = null!;
    
    /// <summary>
    /// Creates a new instance of the application database context
    /// </summary>
    /// <param name="options">DbContext configuration options</param>
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
    {
    }
    
    /// <summary>
    /// Configures the database model using fluent API
    /// </summary>
    /// <param name="modelBuilder">Model builder for configuring entity relationships and constraints</param>
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // Configure Prompt entity
        modelBuilder.Entity<Prompt>(entity => 
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Name).HasMaxLength(100).IsRequired();
            entity.Property(e => e.Description).HasMaxLength(500);
            entity.Property(e => e.Template).IsRequired();
            entity.HasIndex(e => e.Name).IsUnique();
            
            // Configure default values
            entity.Property(e => e.Temperature).HasDefaultValue(0.7m);
            entity.Property(e => e.MaxTokens).HasDefaultValue(1000);
        });
        
        // Configure Completion entity
        modelBuilder.Entity<Completion>(entity => 
        {
            entity.HasKey(e => e.Id);
            
            // Define relationship with Prompt
            entity.HasOne(e => e.Prompt)
                  .WithMany()
                  .HasForeignKey(e => e.PromptId)
                  .OnDelete(DeleteBehavior.Restrict);
                  
            // Store parameters as JSON
            entity.Property(e => e.Parameters)
                  .HasColumnType("jsonb");
        });
        
        // Configure Feedback entity
        modelBuilder.Entity<Feedback>(entity => 
        {
            entity.HasKey(e => e.Id);
            
            // Define relationship with Completion
            entity.HasOne(e => e.Completion)
                  .WithMany()
                  .HasForeignKey(e => e.CompletionId)
                  .OnDelete(DeleteBehavior.Cascade);
        });
        
        // Seed initial data
        SeedData(modelBuilder);
    }
    
    /// <summary>
    /// Seeds initial data for development and testing
    /// </summary>
    /// <param name="modelBuilder">Model builder for configuring seed data</param>
    private void SeedData(ModelBuilder modelBuilder)
    {
        // Seed common prompt templates
        modelBuilder.Entity<Prompt>().HasData(
            new Prompt
            {
                Id = Guid.Parse("8f7b968a-9d6d-4b4c-8e1d-c8f765324dad"),
                Name = "ContentSummarizer",
                Description = "Summarizes content in a concise way",
                Template = "Summarize the following content in {{length}} sentences:\n{{content}}",
                ModelId = "gpt-4",
                DefaultParameters = new Dictionary<string, string> { { "length", "3" } }
            },
            new Prompt
            {
                Id = Guid.Parse("d5e9e01c-1f4c-4c4a-84e2-2b59629e6244"),
                Name = "CodeExplainer",
                Description = "Explains code snippets",
                Template = "Explain the following {{language}} code:\n```{{language}}\n{{code}}\n```",
                ModelId = "gpt-4",
                DefaultParameters = new Dictionary<string, string> { { "language", "csharp" } }
            }
        );
    }
}

---
# Comprehensive Technical Documentation: Building AI-Powered Applications with .NET Aspire and Semantic Kernel (continued)

## Data Access and Repository Pattern

### Entity Framework Core Context

The Data Access Layer uses EF Core to interact with the database:

```csharp
// AIAspireSolution.DataService/Data/AppDbContext.cs
using Microsoft.EntityFrameworkCore;
using AIAspireSolution.Domain.Models;

namespace AIAspireSolution.DataService.Data;

/// <summary>
/// Application database context providing access to all entity collections
/// and configuration for Entity Framework Core
/// </summary>
public class AppDbContext : DbContext
{
    /// <summary>
    /// Collection of AI prompts stored in the database
    /// </summary>
    public DbSet<Prompt> Prompts { get; set; } = null!;
    
    /// <summary>
    /// Collection of AI completions/responses with usage tracking
    /// </summary>
    public DbSet<Completion> Completions { get; set; } = null!;
    
    /// <summary>
    /// Collection of user feedback on AI-generated content
    /// </summary>
    public DbSet<Feedback> Feedback { get; set; } = null!;
    
    /// <summary>
    /// Creates a new instance of the application database context
    /// </summary>
    /// <param name="options">DbContext configuration options</param>
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
    {
    }
    
    /// <summary>
    /// Configures the database model using fluent API
    /// </summary>
    /// <param name="modelBuilder">Model builder for configuring entity relationships and constraints</param>
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // Configure Prompt entity
        modelBuilder.Entity<Prompt>(entity => 
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Name).HasMaxLength(100).IsRequired();
            entity.Property(e => e.Description).HasMaxLength(500);
            entity.Property(e => e.Template).IsRequired();
            entity.HasIndex(e => e.Name).IsUnique();
            
            // Configure default values
            entity.Property(e => e.Temperature).HasDefaultValue(0.7m);
            entity.Property(e => e.MaxTokens).HasDefaultValue(1000);
        });
        
        // Configure Completion entity
        modelBuilder.Entity<Completion>(entity => 
        {
            entity.HasKey(e => e.Id);
            
            // Define relationship with Prompt
            entity.HasOne(e => e.Prompt)
                  .WithMany()
                  .HasForeignKey(e => e.PromptId)
                  .OnDelete(DeleteBehavior.Restrict);
                  
            // Store parameters as JSON
            entity.Property(e => e.Parameters)
                  .HasColumnType("jsonb");
        });
        
        // Configure Feedback entity
        modelBuilder.Entity<Feedback>(entity => 
        {
            entity.HasKey(e => e.Id);
            
            // Define relationship with Completion
            entity.HasOne(e => e.Completion)
                  .WithMany()
                  .HasForeignKey(e => e.CompletionId)
                  .OnDelete(DeleteBehavior.Cascade);
        });
        
        // Seed initial data
        SeedData(modelBuilder);
    }
    
    /// <summary>
    /// Seeds initial data for development and testing
    /// </summary>
    /// <param name="modelBuilder">Model builder for configuring seed data</param>
    private void SeedData(ModelBuilder modelBuilder)
    {
        // Seed common prompt templates
        modelBuilder.Entity<Prompt>().HasData(
            new Prompt
            {
                Id = Guid.Parse("8f7b968a-9d6d-4b4c-8e1d-c8f765324dad"),
                Name = "ContentSummarizer",
                Description = "Summarizes content in a concise way",
                Template = "Summarize the following content in {{length}} sentences:\n{{content}}",
                ModelId = "gpt-4",
                DefaultParameters = new Dictionary<string, string> { { "length", "3" } }
            },
            new Prompt
            {
                Id = Guid.Parse("d5e9e01c-1f4c-4c4a-84e2-2b59629e6244"),
                Name = "CodeExplainer",
                Description = "Explains code snippets",
                Template = "Explain the following {{language}} code:\n```{{language}}\n{{code}}\n```",
                ModelId = "gpt-4",
                DefaultParameters = new Dictionary<string, string> { { "language", "csharp" } }
            }
        );
    }
}
```

### Generic Repository Implementation

```csharp
// AIAspireSolution.DataService/Repositories/GenericRepository.cs
using System.Linq.Expressions;
using Microsoft.EntityFrameworkCore;
using AIAspireSolution.Domain.Interfaces;
using AIAspireSolution.DataService.Data;

namespace AIAspireSolution.DataService.Repositories;

/// <summary>
/// Generic repository implementation providing CRUD operations for any entity type
/// </summary>
/// <typeparam name="TEntity">Entity type this repository manages</typeparam>
public class GenericRepository<TEntity> : IGenericRepository<TEntity> where TEntity : class
{
    /// <summary>
    /// Database context for entity access
    /// </summary>
    protected readonly AppDbContext _context;
    
    /// <summary>
    /// DbSet for the specific entity type
    /// </summary>
    protected readonly DbSet<TEntity> _dbSet;
    
    /// <summary>
    /// Creates a new repository instance
    /// </summary>
    /// <param name="context">Database context</param>
    public GenericRepository(AppDbContext context)
    {
        _context = context;
        _dbSet = context.Set<TEntity>();
    }
    
    /// <summary>
    /// Retrieves all entities of type TEntity from the database
    /// </summary>
    /// <returns>Collection of all entities</returns>
    public virtual async Task<IEnumerable<TEntity>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }
    
    /// <summary>
    /// Finds entities matching a specified condition
    /// </summary>
    /// <param name="predicate">Filter expression</param>
    /// <returns>Collection of matching entities</returns>
    public virtual async Task<IEnumerable<TEntity>> FindAsync(Expression<Func<TEntity, bool>> predicate)
    {
        return await _dbSet.Where(predicate).ToListAsync();
    }
    
    /// <summary>
    /// Retrieves an entity by its primary key
    /// </summary>
    /// <param name="id">Primary key value</param>
    /// <returns>Entity with matching ID or null</returns>
    public virtual async Task<TEntity?> GetByIdAsync(object id)
    {
        return await _dbSet.FindAsync(id);
    }
    
    /// <summary>
    /// Retrieves a single entity matching the provided condition
    /// </summary>
    /// <param name="predicate">Filter expression</param>
    /// <returns>Single matching entity or null</returns>
    public virtual async Task<TEntity?> SingleOrDefaultAsync(Expression<Func<TEntity, bool>> predicate)
    {
        return await _dbSet.SingleOrDefaultAsync(predicate);
    }
    
    /// <summary>
    /// Adds a new entity to the repository
    /// </summary>
    /// <param name="entity">Entity to add</param>
    public virtual async Task AddAsync(TEntity entity)
    {
        await _dbSet.AddAsync(entity);
    }
    
    /// <summary>
    /// Adds multiple entities to the repository
    /// </summary>
    /// <param name="entities">Collection of entities to add</param>
    public virtual async Task AddRangeAsync(IEnumerable<TEntity> entities)
    {
        await _dbSet.AddRangeAsync(entities);
    }
    
    /// <summary>
    /// Updates an existing entity in the repository
    /// </summary>
    /// <param name="entity">Entity to update</param>
    public virtual void Update(TEntity entity)
    {
        _dbSet.Attach(entity);
        _context.Entry(entity).State = EntityState.Modified;
    }
    
    /// <summary>
    /// Removes an entity from the repository
    /// </summary>
    /// <param name="entity">Entity to remove</param>
    public virtual void Remove(TEntity entity)
    {
        if (_context.Entry(entity).State == EntityState.Detached)
        {
            _dbSet.Attach(entity);
        }
        _dbSet.Remove(entity);
    }
    
    /// <summary>
    /// Removes multiple entities from the repository
    /// </summary>
    /// <param name="entities">Collection of entities to remove</param>
    public virtual void RemoveRange(IEnumerable<TEntity> entities)
    {
        _dbSet.RemoveRange(entities);
    }
}
```

### Unit of Work Pattern

```csharp
// AIAspireSolution.DataService/UnitOfWork/UnitOfWork.cs
using AIAspireSolution.Domain.Interfaces;
using AIAspireSolution.DataService.Data;
using AIAspireSolution.DataService.Repositories;
using AIAspireSolution.Domain.Models;

namespace AIAspireSolution.DataService.UnitOfWork;

/// <summary>
/// Unit of Work implementation that coordinates operations across multiple repositories
/// and manages database transactions
/// </summary>
public class UnitOfWork : IUnitOfWork
{
    /// <summary>
    /// Database context shared by all repositories
    /// </summary>
    private readonly AppDbContext _context;
    
    /// <summary>
    /// Lazy-loaded repository for prompts
    /// </summary>
    private IGenericRepository<Prompt>? _promptsRepository;
    
    /// <summary>
    /// Lazy-loaded repository for completions
    /// </summary>
    private IGenericRepository<Completion>? _completionsRepository;
    
    /// <summary>
    /// Lazy-loaded repository for feedback
    /// </summary>
    private IGenericRepository<Feedback>? _feedbackRepository;
    
    /// <summary>
    /// Lazy-loaded specialized prompt repository
    /// </summary>
    private IPromptRepository? _promptRepository;
    
    /// <summary>
    /// Creates a new Unit of Work instance
    /// </summary>
    /// <param name="context">Database context to use</param>
    public UnitOfWork(AppDbContext context)
    {
        _context = context;
    }
    
    /// <summary>
    /// Gets a repository for prompt entities
    /// </summary>
    public IGenericRepository<Prompt> Prompts => 
        _promptsRepository ??= new GenericRepository<Prompt>(_context);
    
    /// <summary>
    /// Gets a repository for completion entities
    /// </summary>
    public IGenericRepository<Completion> Completions => 
        _completionsRepository ??= new GenericRepository<Completion>(_context);
    
    /// <summary>
    /// Gets a repository for feedback entities
    /// </summary>
    public IGenericRepository<Feedback> Feedback => 
        _feedbackRepository ??= new GenericRepository<Feedback>(_context);
    
    /// <summary>
    /// Gets a specialized repository for prompt operations
    /// </summary>
    public IPromptRepository PromptRepository => 
        _promptRepository ??= new PromptRepository(_context);
    
    /// <summary>
    /// Saves all changes made through the repositories to the database
    /// </summary>
    /// <returns>Number of affected database rows</returns>
    public async Task<int> CompleteAsync()
    {
        return await _context.SaveChangesAsync();
    }
    
    /// <summary>
    /// Disposes the database context and resources
    /// </summary>
    public void Dispose()
    {
        _context.Dispose();
    }
}
```

### Specialized Repository

```csharp
// AIAspireSolution.DataService/Repositories/PromptRepository.cs
using Microsoft.EntityFrameworkCore;
using AIAspireSolution.Domain.Interfaces;
using AIAspireSolution.Domain.Models;
using AIAspireSolution.DataService.Data;

namespace AIAspireSolution.DataService.Repositories;

/// <summary>
/// Specialized repository for Prompt entities with prompt-specific operations
/// </summary>
public class PromptRepository : GenericRepository<Prompt>, IPromptRepository
{
    /// <summary>
    /// Creates a new prompt repository instance
    /// </summary>
    /// <param name="context">Database context</param>
    public PromptRepository(AppDbContext context) : base(context)
    {
    }
    
    /// <summary>
    /// Finds prompts by their name using case-insensitive search
    /// </summary>
    /// <param name="name">Prompt name to search for</param>
    /// <returns>Prompt with matching name or null</returns>
    public async Task<Prompt?> GetByNameAsync(string name)
    {
        // Case-insensitive search by name
        return await _dbSet
            .FirstOrDefaultAsync(p => EF.Functions.ILike(p.Name, name));
    }
    
    /// <summary>
    /// Finds prompts by category
    /// </summary>
    /// <param name="category">Category to search for</param>
    /// <returns>Collection of prompts in the specified category</returns>
    public async Task<IEnumerable<Prompt>> GetByCategoryAsync(string category)
    {
        return await _dbSet
            .Where(p => p.Category == category)
            .OrderBy(p => p.Name)
            .ToListAsync();
    }
    
    /// <summary>
    /// Searches prompts by tag (stored as comma-separated values)
    /// </summary>
    /// <param name="tag">Tag to search for</param>
    /// <returns>Collection of prompts with the specified tag</returns>
    public async Task<IEnumerable<Prompt>> GetByTagAsync(string tag)
    {
        // Search for tag in the comma-separated Tags field
        return await _dbSet
            .Where(p => p.Tags.Contains(tag))
            .OrderBy(p => p.Name)
            .ToListAsync();
    }
    
    /// <summary>
    /// Gets all available prompts with pagination support
    /// </summary>
    /// <param name="skip">Number of items to skip</param>
    /// <param name="take">Maximum number of items to take</param>
    /// <returns>Collection of prompts with pagination</returns>
    public async Task<IEnumerable<Prompt>> GetPagedPromptsAsync(int skip, int take)
    {
        return await _dbSet
            .OrderBy(p => p.Name)
            .Skip(skip)
            .Take(take)
            .ToListAsync();
    }
    
    /// <summary>
    /// Gets statistics about prompt usage from completions
    /// </summary>
    /// <returns>Collection of prompt usage statistics</returns>
    public async Task<IEnumerable<PromptUsageStatistic>> GetPromptUsageStatisticsAsync()
    {
        // Get completion statistics grouped by prompt
        var stats = await _context.Completions
            .GroupBy(c => c.PromptId)
            .Select(g => new PromptUsageStatistic
            {
                PromptId = g.Key,
                UsageCount = g.Count(),
                TotalTokens = g.Sum(c => c.TokensUsed),
                AverageLatencyMs = g.Average(c => c.LatencyMs.TotalMilliseconds)
            })
            .ToListAsync();
            
        // Merge with prompt names
        foreach (var stat in stats)
        {
            var prompt = await _dbSet.FindAsync(stat.PromptId);
            if (prompt != null)
            {
                stat.PromptName = prompt.Name;
            }
        }
        
        return stats;
    }
}

/// <summary>
/// Statistics about prompt usage
/// </summary>
public class PromptUsageStatistic
{
    /// <summary>
    /// ID of the prompt
    /// </summary>
    public Guid PromptId { get; set; }
    
    /// <summary>
    /// Name of the prompt
    /// </summary>
    public string PromptName { get; set; } = string.Empty;
    
    /// <summary>
    /// Number of times the prompt was used
    /// </summary>
    public int UsageCount { get; set; }
    
    /// <summary>
    /// Total tokens consumed by this prompt
    /// </summary>
    public int TotalTokens { get; set; }
    
    /// <summary>
    /// Average latency for completions using this prompt (milliseconds)
    /// </summary>
    public double AverageLatencyMs { get; set; }
}
```

## Prompt Engineering & Management

### Prompt Service Implementation

```csharp
// AIAspireSolution.DataService/Services/PromptService.cs
using System.Text.RegularExpressions;
using AIAspireSolution.Domain.Interfaces;
using AIAspireSolution.Domain.Models;
using Microsoft.SemanticKernel;
using Microsoft.Extensions.Logging;

namespace AIAspireSolution.DataService.Services;

/// <summary>
/// Service for managing and processing prompt templates
/// </summary>
public class PromptService : IPromptService
{
    /// <summary>
    /// Unit of work for database operations
    /// </summary>
    private readonly IUnitOfWork _unitOfWork;
    
    /// <summary>
    /// Logger for service operations
    /// </summary>
    private readonly ILogger<PromptService> _logger;
    
    /// <summary>
    /// Regular expression for finding template variables
    /// </summary>
    private static readonly Regex _variableRegex = new Regex(@"\{\{([^}]+)\}\}", RegexOptions.Compiled);
    
    /// <summary>
    /// Creates a new prompt service
    /// </summary>
    /// <param name="unitOfWork">Unit of work for database operations</param>
    /// <param name="logger">Logger for service operations</param>
    public PromptService(IUnitOfWork unitOfWork, ILogger<PromptService> logger)
    {
        _unitOfWork = unitOfWork;
        _logger = logger;
    }

    /// <summary>
    /// Gets all available prompts
    /// </summary>
    /// <returns>Collection of all prompts</returns>
    public async Task<IEnumerable<Prompt>> GetAllPromptsAsync()
    {
        return await _unitOfWork.Prompts.GetAllAsync();
    }

    /// <summary>
    /// Gets a prompt by its ID
    /// </summary>
    /// <param name="id">Prompt ID to find</param>
    /// <returns>Matching prompt or null</returns>
    public async Task<Prompt?> GetPromptAsync(Guid id)
    {
        return await _unitOfWork.Prompts.GetByIdAsync(id);
    }

    /// <summary>
    /// Gets a prompt by its name
    /// </summary>
    /// <param name="name">Name of the prompt to find</param>
    /// <returns>Matching prompt or null</returns>
    public async Task<Prompt?> GetPromptByNameAsync(string name)
    {
        return await _unitOfWork.PromptRepository.GetByNameAsync(name);
    }

    /// <summary>
    /// Gets prompts by category
    /// </summary>
    /// <param name="category">Category to filter by</param>
    /// <returns>Collection of prompts in the category</returns>
    public async Task<IEnumerable<Prompt>> GetPromptsByCategoryAsync(string category)
    {
        return await _unitOfWork.PromptRepository.GetByCategoryAsync(category);
    }

    /// <summary>
    /// Gets prompts containing any of the specified tags
    /// </summary>
    /// <param name="tags">Tags to search for</param>
    /// <returns>Collection of prompts with matching tags</returns>
    public async Task<IEnumerable<Prompt>> GetPromptsByTagsAsync(string[] tags)
    {
        // Get prompts for each tag and combine them, removing duplicates
        var results = new List<Prompt>();
        foreach (var tag in tags)
        {
            var taggedPrompts = await _unitOfWork.PromptRepository.GetByTagAsync(tag);
            foreach (var prompt in taggedPrompts)
            {
                // Only add if not already in results
                if (!results.Any(p => p.Id == prompt.Id))
                {
                    results.Add(prompt);
                }
            }
        }
        
        return results;
    }

    /// <summary>
    /// Creates a new prompt
    /// </summary>
    /// <param name="prompt">Prompt to create</param>
    /// <returns>Created prompt with ID</returns>
    public async Task<Prompt> CreatePromptAsync(Prompt prompt)
    {
        // Validate template for syntax errors
        ValidatePromptTemplate(prompt.Template);
        
        // Set creation time
        prompt.CreatedAt = DateTime.UtcNow;
        
        // Add to repository
        await _unitOfWork.Prompts.AddAsync(prompt);
        await _unitOfWork.CompleteAsync();
        
        return prompt;
    }

    /// <summary>
    /// Updates an existing prompt
    /// </summary>
    /// <param name="prompt">Prompt with updated values</param>
    /// <returns>Updated prompt</returns>
    public async Task<Prompt> UpdatePromptAsync(Prompt prompt)
    {
        // Validate template for syntax errors
        ValidatePromptTemplate(prompt.Template);
        
        // Update modification time
        prompt.UpdatedAt = DateTime.UtcNow;
        
        // Increment version
        prompt.Version += 1;
        
        // Update in repository
        _unitOfWork.Prompts.Update(prompt);
        await _unitOfWork.CompleteAsync();
        
        return prompt;
    }

    /// <summary>
    /// Deletes a prompt by ID
    /// </summary>
    /// <param name="id">ID of prompt to delete</param>
    /// <returns>True if deleted, false if not found</returns>
    public async Task<bool> DeletePromptAsync(Guid id)
    {
        var prompt = await _unitOfWork.Prompts.GetByIdAsync(id);
        if (prompt == null)
        {
            return false;
        }
        
        _unitOfWork.Prompts.Remove(prompt);
        await _unitOfWork.CompleteAsync();
        
        return true;
    }

    /// <summary>
    /// Renders a prompt template by name with provided arguments
    /// </summary>
    /// <param name="promptName">Name of the prompt template</param>
    /// <param name="arguments">Arguments to use when rendering</param>
    /// <returns>Rendered prompt ready for model submission</returns>
    public async Task<string> ProcessPromptTemplateAsync(string promptName, KernelArguments arguments)
    {
        // Find the prompt by name
        var prompt = await _unitOfWork.PromptRepository.GetByNameAsync(promptName);
        if (prompt == null)
        {
            throw new KeyNotFoundException($"Prompt template '{promptName}' not found");
        }
        
        return await ProcessPromptTemplateByIdAsync(prompt.Id, arguments);
    }

    /// <summary>
    /// Renders a prompt template by ID with provided arguments
    /// </summary>
    /// <param name="promptId">ID of the prompt template</param>
    /// <param name="arguments">Arguments to use when rendering</param>
    /// <returns>Rendered prompt ready for model submission</returns>
    public async Task<string> ProcessPromptTemplateByIdAsync(Guid promptId, KernelArguments arguments)
    {
        // Find the prompt by ID
        var prompt = await _unitOfWork.Prompts.GetByIdAsync(promptId);
        if (prompt == null)
        {
            throw new KeyNotFoundException($"Prompt template with ID '{promptId}' not found");
        }
        
        _logger.LogDebug("Processing prompt template: {PromptName}", prompt.Name);
        
        // Get the template
        var template = prompt.Template;
        
        // Apply default parameters for missing values
        if (prompt.DefaultParameters != null)
        {
            foreach (var param in prompt.DefaultParameters)
            {
                if (!arguments.ContainsKey(param.Key))
                {
                    arguments[param.Key] = param.Value;
                }
            }
        }
        
        // Process the template by replacing variables
        var renderedPrompt = _variableRegex.Replace(template, match =>
        {
            var varName = match.Groups[1].Value.Trim();
            
            if (arguments.ContainsKey(varName))
            {
                return arguments[varName]?.ToString() ?? string.Empty;
            }
            
            _logger.LogWarning("Missing argument '{VarName}' for prompt '{PromptName}'", varName, prompt.Name);
            return $"[MISSING: {varName}]";
        });
        
        return renderedPrompt;
    }
    
    /// <summary>
    /// Validates a prompt template for syntax errors
    /// </summary>
    /// <param name="template">Template to validate</param>
    /// <exception cref="ArgumentException">Thrown if template contains syntax errors</exception>
    private void ValidatePromptTemplate(string template)
    {
        if (string.IsNullOrEmpty(template))
        {
            throw new ArgumentException("Prompt template cannot be empty");
        }
        
        // Ensure all {{ have matching }}
        var openCount = Regex.Matches(template, "\\{\\{").Count;
        var closeCount = Regex.Matches(template, "\\}\\}").Count;
        
        if (openCount != closeCount)
        {
            throw new ArgumentException("Prompt template has mismatched {{ and }} delimiters");
        }
        
        // Ensure all variables have a name
        var matches = _variableRegex.Matches(template);
        foreach (Match match in matches)
        {
            if (string.IsNullOrWhiteSpace(match.Groups[1].Value))
            {
                throw new ArgumentException("Prompt template contains empty variable name: {{}}");
            }
        }
    }
}
```

### Completion Tracking Service

```csharp
// AIAspireSolution.DataService/Services/CompletionService.cs
using AIAspireSolution.Domain.Interfaces;
using AIAspireSolution.Domain.Models;
using Microsoft.Extensions.Logging;

namespace AIAspireSolution.DataService.Services;

/// <summary>
/// Service for tracking and managing AI completions
/// </summary>
public class CompletionService : ICompletionService
{
    /// <summary>
    /// Unit of work for database operations
    /// </summary>
    private readonly IUnitOfWork _unitOfWork;
    
    /// <summary>
    /// Logger for service operations
    /// </summary>
    private readonly ILogger<CompletionService> _logger;
    
    /// <summary>
    /// Creates a new completion service
    /// </summary>
    /// <param name="unitOfWork">Unit of work for database operations</param>
    /// <param name="logger">Logger for service operations</param>
    public CompletionService(IUnitOfWork unitOfWork, ILogger<CompletionService> logger)
    {
        _unitOfWork = unitOfWork;
        _logger = logger;
    }

    /// <summary>
    /// Tracks a new completion from an AI model
    /// </summary>
    /// <param name="completion">Completion to track</param>
    /// <returns>Tracked completion with ID</returns>
    public async Task<Completion> TrackCompletionAsync(Completion completion)
    {
        _logger.LogDebug("Tracking completion for prompt ID: {PromptId}", completion.PromptId);
        
        // Set creation time if not already set
        if (completion.CreatedAt == default)
        {
            completion.CreatedAt = DateTime.UtcNow;
        }
        
        // Add to repository
        await _unitOfWork.Completions.AddAsync(completion);
        await _unitOfWork.CompleteAsync();
        
        return completion;
    }
    
    /// <summary>
    /// Gets a completion by ID
    /// </summary>
    /// <param name="id">Completion ID</param>
    /// <returns>Matching completion or null</returns>
    public async Task<Completion?> GetCompletionAsync(Guid id)
    {
        return await _unitOfWork.Completions.GetByIdAsync(id);
    }

    /// <summary>
    /// Gets all completions for a specific prompt
    /// </summary>
    /// <param name="promptId">ID of the prompt</param>
    /// <returns>Collection of completions for the prompt</returns>
    public async Task<IEnumerable<Completion>> GetCompletionsByPromptAsync(Guid promptId)
    {
        return await _unitOfWork.Completions.FindAsync(c => c.PromptId == promptId);
    }

    /// <summary>
    /// Gets feedback for a completion
    /// </summary>
    /// <param name="completionId">ID of the completion</param>
    /// <returns>Collection of feedback items for the completion</returns>
    public async Task<IEnumerable<Feedback>> GetFeedbackForCompletionAsync(Guid completionId)
    {
        return await _unitOfWork.Feedback.FindAsync(f => f.CompletionId == completionId);
    }

    /// <summary>
    /// Adds feedback for a completion
    /// </summary>
    /// <param name="feedback">Feedback to add</param>
    /// <returns>Added feedback with ID</returns>
    public async Task<Feedback> AddFeedbackAsync(Feedback feedback)
    {
        // Set creation time if not already set
        if (feedback.CreatedAt == default)
        {
            feedback.CreatedAt = DateTime.UtcNow;
        }
        
        // Add to repository
        await _unitOfWork.Feedback.AddAsync(feedback);
        await _unitOfWork.CompleteAsync();
        
        return feedback;
    }
    
    /// <summary>
    /// Gets statistics about completion usage
    /// </summary>
    /// <param name="days">Number of days to include in statistics</param>
    /// <returns>Usage statistics for the specified period</returns>
    public async Task<CompletionStatistics> GetCompletionStatisticsAsync(int days = 30)
    {
        var cutoffDate = DateTime.UtcNow.AddDays(-days);
        
        // Get all completions in the period
        var completions = await _unitOfWork.Completions.FindAsync(c => c.CreatedAt >= cutoffDate);
        var completionsList = completions.ToList();
        
        // Calculate statistics
        return new CompletionStatistics
        {
            TotalCompletions = completionsList.Count,
            TotalTokens = completionsList.Sum(c => c.TokensUsed),
            AverageTokens = completion
        
        }
        ....
---

# Comprehensive Technical Documentation: Building AI-Powered Applications with .NET Aspire and Semantic Kernel (continued)

**Current Date: 2025-03-17**

```csharp
// AIAspireSolution.DataService/Services/CompletionService.cs (continued)
    public async Task<CompletionStatistics> GetCompletionStatisticsAsync(int days = 30)
    {
        var cutoffDate = DateTime.UtcNow.AddDays(-days);
        
        // Get all completions in the period
        var completions = await _unitOfWork.Completions.FindAsync(c => c.CreatedAt >= cutoffDate);
        var completionsList = completions.ToList();
        
        // Calculate statistics
        return new CompletionStatistics
        {
            TotalCompletions = completionsList.Count,
            TotalTokens = completionsList.Sum(c => c.TokensUsed),
            AverageTokens = completionsList.Count > 0 ? completionsList.Average(c => c.TokensUsed) : 0,
            TotalCost = completionsList.Sum(c => c.Cost),
            AverageLatencyMs = completionsList.Count > 0 ? completionsList.Average(c => c.LatencyMs.TotalMilliseconds) : 0,
            CompletionsByDay = completionsList
                .GroupBy(c => c.CreatedAt.Date)
                .Select(g => new DailyCompletionStats 
                {
                    Date = g.Key,
                    Count = g.Count(),
                    Tokens = g.Sum(c => c.TokensUsed)
                })
                .OrderBy(s => s.Date)
                .ToList(),
            ModelUsage = completionsList
                .GroupBy(c => c.ModelId)
                .Select(g => new ModelUsageStats
                {
                    ModelId = g.Key,
                    Count = g.Count(),
                    Tokens = g.Sum(c => c.TokensUsed),
                    Cost = g.Sum(c => c.Cost)
                })
                .OrderByDescending(s => s.Count)
                .ToList()
        };
    }
}

/// <summary>
/// Statistics about completions over a period of time
/// </summary>
public class CompletionStatistics
{
    /// <summary>
    /// Total number of completions in the period
    /// </summary>
    public int TotalCompletions { get; set; }
    
    /// <summary>
    /// Total tokens used across all completions
    /// </summary>
    public int TotalTokens { get; set; }
    
    /// <summary>
    /// Average tokens per completion
    /// </summary>
    public double AverageTokens { get; set; }
    
    /// <summary>
    /// Total cost of all completions
    /// </summary>
    public decimal TotalCost { get; set; }
    
    /// <summary>
    /// Average latency in milliseconds
    /// </summary>
    public double AverageLatencyMs { get; set; }
    
    /// <summary>
    /// Daily breakdown of completions
    /// </summary>
    public List<DailyCompletionStats> CompletionsByDay { get; set; } = new();
    
    /// <summary>
    /// Usage statistics by model
    /// </summary>
    public List<ModelUsageStats> ModelUsage { get; set; } = new();
}

/// <summary>
/// Daily completion statistics
/// </summary>
public class DailyCompletionStats
{
    /// <summary>
    /// Date for these statistics
    /// </summary>
    public DateTime Date { get; set; }
    
    /// <summary>
    /// Number of completions on this date
    /// </summary>
    public int Count { get; set; }
    
    /// <summary>
    /// Tokens used on this date
    /// </summary>
    public int Tokens { get; set; }
}

/// <summary>
/// Usage statistics for a specific model
/// </summary>
public class ModelUsageStats
{
    /// <summary>
    /// Model identifier
    /// </summary>
    public string ModelId { get; set; } = string.Empty;
    
    /// <summary>
    /// Number of completions with this model
    /// </summary>
    public int Count { get; set; }
    
    /// <summary>
    /// Tokens used with this model
    /// </summary>
    public int Tokens { get; set; }
    
    /// <summary>
    /// Total cost for this model
    /// </summary>
    public decimal Cost { get; set; }
}
```

## Testing Strategy

### Unit Testing Services

```csharp
// AIAspireSolution.AiService.Tests/Services/PromptServiceTests.cs
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using AIAspireSolution.Domain.Interfaces;
using AIAspireSolution.Domain.Models;
using AIAspireSolution.DataService.Services;
using Microsoft.Extensions.Logging;
using Microsoft.SemanticKernel;
using Moq;
using Xunit;

namespace AIAspireSolution.AiService.Tests.Services;

/// <summary>
/// Unit tests for the PromptService class
/// </summary>
public class PromptServiceTests
{
    /// <summary>
    /// Mock unit of work for testing
    /// </summary>
    private readonly Mock<IUnitOfWork> _mockUnitOfWork;
    
    /// <summary>
    /// Mock prompt repository for testing
    /// </summary>
    private readonly Mock<IPromptRepository> _mockPromptRepository;
    
    /// <summary>
    /// Mock generic repository for prompts
    /// </summary>
    private readonly Mock<IGenericRepository<Prompt>> _mockGenericRepository;
    
    /// <summary>
    /// Mock logger for testing
    /// </summary>
    private readonly Mock<ILogger<PromptService>> _mockLogger;
    
    /// <summary>
    /// Service under test
    /// </summary>
    private readonly PromptService _promptService;

    /// <summary>
    /// Sets up test fixtures and dependencies
    /// </summary>
    public PromptServiceTests()
    {
        // Create mocks
        _mockUnitOfWork = new Mock<IUnitOfWork>();
        _mockPromptRepository = new Mock<IPromptRepository>();
        _mockGenericRepository = new Mock<IGenericRepository<Prompt>>();
        _mockLogger = new Mock<ILogger<PromptService>>();
        
        // Setup unit of work
        _mockUnitOfWork.Setup(u => u.PromptRepository).Returns(_mockPromptRepository.Object);
        _mockUnitOfWork.Setup(u => u.Prompts).Returns(_mockGenericRepository.Object);
        
        // Create service instance
        _promptService = new PromptService(_mockUnitOfWork.Object, _mockLogger.Object);
    }

    /// <summary>
    /// Tests that GetPromptByNameAsync returns the correct prompt
    /// </summary>
    [Fact]
    public async Task GetPromptByNameAsync_ExistingPrompt_ReturnsPrompt()
    {
        // Arrange
        var expectedPrompt = new Prompt
        {
            Id = Guid.NewGuid(),
            Name = "TestPrompt",
            Template = "This is a test prompt with {{parameter}}."
        };
        
        _mockPromptRepository.Setup(r => r.GetByNameAsync("TestPrompt"))
                             .ReturnsAsync(expectedPrompt);
        
        // Act
        var result = await _promptService.GetPromptByNameAsync("TestPrompt");
        
        // Assert
        Assert.NotNull(result);
        Assert.Equal(expectedPrompt.Id, result.Id);
        Assert.Equal(expectedPrompt.Name, result.Name);
    }

    /// <summary>
    /// Tests that ProcessPromptTemplateAsync correctly fills in template parameters
    /// </summary>
    [Fact]
    public async Task ProcessPromptTemplateAsync_ValidTemplate_ReplacesParameters()
    {
        // Arrange
        var promptId = Guid.NewGuid();
        var prompt = new Prompt
        {
            Id = promptId,
            Name = "TestPrompt",
            Template = "Hello {{name}}! Your favorite color is {{color}}."
        };
        
        _mockPromptRepository.Setup(r => r.GetByNameAsync("TestPrompt"))
                             .ReturnsAsync(prompt);
        
        var arguments = new KernelArguments
        {
            ["name"] = "John",
            ["color"] = "blue"
        };
        
        // Act
        var result = await _promptService.ProcessPromptTemplateAsync("TestPrompt", arguments);
        
        // Assert
        Assert.Equal("Hello John! Your favorite color is blue.", result);
    }

    /// <summary>
    /// Tests that ProcessPromptTemplateAsync uses default parameters when not provided
    /// </summary>
    [Fact]
    public async Task ProcessPromptTemplateAsync_MissingParameter_UsesDefaultValue()
    {
        // Arrange
        var promptId = Guid.NewGuid();
        var prompt = new Prompt
        {
            Id = promptId,
            Name = "GreetingPrompt",
            Template = "Hello {{name}}! Welcome to {{service}}.",
            DefaultParameters = new Dictionary<string, string>
            {
                { "service", "AI Aspire Service" }
            }
        };
        
        _mockPromptRepository.Setup(r => r.GetByNameAsync("GreetingPrompt"))
                             .ReturnsAsync(prompt);
        
        var arguments = new KernelArguments
        {
            ["name"] = "Jane"
            // Note: 'service' parameter is missing but has a default value
        };
        
        // Act
        var result = await _promptService.ProcessPromptTemplateAsync("GreetingPrompt", arguments);
        
        // Assert
        Assert.Equal("Hello Jane! Welcome to AI Aspire Service.", result);
    }

    /// <summary>
    /// Tests that CreatePromptAsync validates template syntax
    /// </summary>
    [Fact]
    public async Task CreatePromptAsync_InvalidTemplate_ThrowsArgumentException()
    {
        // Arrange
        var invalidPrompt = new Prompt
        {
            Name = "InvalidPrompt",
            Template = "This has an unclosed {{ bracket"
        };
        
        // Act & Assert
        await Assert.ThrowsAsync<ArgumentException>(() => 
            _promptService.CreatePromptAsync(invalidPrompt));
    }
}
```

### Integration Testing

```csharp
// AIAspireSolution.IntegrationTests/AiService/SemanticKernelIntegrationTests.cs
using System;
using System.Threading.Tasks;
using AIAspireSolution.AiService.Planning;
using AIAspireSolution.AiService.Plugins;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.SemanticKernel;
using Xunit;

namespace AIAspireSolution.IntegrationTests.AiService;

/// <summary>
/// Integration tests for Semantic Kernel functionality
/// Requires valid OpenAI API key to be set in appsettings.json or environment variables
/// </summary>
[Collection("Integration Tests")]
public class SemanticKernelIntegrationTests : IDisposable
{
    /// <summary>
    /// Service provider for test dependencies
    /// </summary>
    private readonly ServiceProvider _serviceProvider;
    
    /// <summary>
    /// Semantic Kernel instance for testing
    /// </summary>
    private readonly Kernel _kernel;

    /// <summary>
    /// Sets up the test environment
    /// </summary>
    public SemanticKernelIntegrationTests()
    {
        // Create configuration
        var configuration = new ConfigurationBuilder()
            .AddJsonFile("appsettings.json", optional: true)
            .AddUserSecrets<SemanticKernelIntegrationTests>()
            .AddEnvironmentVariables()
            .Build();
            
        // Create service collection
        var services = new ServiceCollection();
        
        // Configure kernel
        services.AddSingleton(sp =>
        {
            var builder = Kernel.CreateBuilder();
            
            // Add OpenAI service
            builder.AddOpenAIChatCompletion(
                modelId: configuration["AI:OpenAI:ModelId"] ?? "gpt-4",
                apiKey: configuration["AI:OpenAI:ApiKey"] ?? throw new InvalidOperationException("OpenAI API key not found")
            );
            
            return builder.Build();
        });
        
        // Build service provider
        _serviceProvider = services.BuildServiceProvider();
        
        // Get kernel
        _kernel = _serviceProvider.GetRequiredService<Kernel>();
        
        // Register plugins
        _kernel.ImportPluginFromObject(new TimePlugin(), "TimePlugin");
    }

    /// <summary>
    /// Tests that the kernel can execute a simple prompt
    /// </summary>
    [Fact]
    public async Task Kernel_ExecutePrompt_ReturnsResult()
    {
        // Arrange
        var prompt = "What is 2 + 2? Answer with just a number.";
        var function = _kernel.CreateFunctionFromPrompt(prompt);
        
        // Act
        var result = await _kernel.InvokeAsync(function);
        var answer = result.GetValue<string>();
        
        // Assert
        Assert.NotNull(answer);
        Assert.Contains("4", answer);
    }

    /// <summary>
    /// Tests that the TimePlugin works correctly
    /// </summary>
    [Fact]
    public async Task Kernel_ExecuteTimePlugin_ReturnsCurrentDate()
    {
        // Arrange
        var prompt = "What is today's date? Use the TimePlugin to find out.";
        var function = _kernel.CreateFunctionFromPrompt(prompt);
        
        // Act
        var result = await _kernel.InvokeAsync(function);
        var answer = result.GetValue<string>();
        
        // Assert
        Assert.NotNull(answer);
        
        // Check that the answer contains today's year
        var currentYear = DateTime.Now.Year.ToString();
        Assert.Contains(currentYear, answer);
    }

    /// <summary>
    /// Disposes test resources
    /// </summary>
    public void Dispose()
    {
        _serviceProvider?.Dispose();
    }
}
```

## Deployment and DevOps

### Dockerfile for Services

```dockerfile
# AIAspireSolution/src/AIAspireSolution.AiService/Dockerfile

# Build stage
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /src

# Copy csproj files and restore dependencies
COPY ["AIAspireSolution.Domain/*.csproj", "AIAspireSolution.Domain/"]
COPY ["AIAspireSolution.AiService/*.csproj", "AIAspireSolution.AiService/"]
COPY ["AIAspireSolution.ServiceDefaults/*.csproj", "AIAspireSolution.ServiceDefaults/"]

# Restore the project dependencies
RUN dotnet restore "AIAspireSolution.AiService/AIAspireSolution.AiService.csproj"

# Copy the rest of the source code
COPY ["AIAspireSolution.Domain/.", "AIAspireSolution.Domain/"]
COPY ["AIAspireSolution.AiService/.", "AIAspireSolution.AiService/"]
COPY ["AIAspireSolution.ServiceDefaults/.", "AIAspireSolution.ServiceDefaults/"]

# Build and publish the project
WORKDIR "/src/AIAspireSolution.AiService"
RUN dotnet build "AIAspireSolution.AiService.csproj" -c Release -o /app/build
RUN dotnet publish "AIAspireSolution.AiService.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS runtime
WORKDIR /app

# Configure environment
ENV ASPNETCORE_ENVIRONMENT=Production
ENV ASPNETCORE_URLS=http://+:80

# Copy build output to runtime image
COPY --from=build /app/publish .

# Set entry point
ENTRYPOINT ["dotnet", "AIAspireSolution.AiService.dll"]
```

### GitHub Actions CI/CD Pipeline

```yaml
# .github/workflows/ai-service-cicd.yml

name: AI Service CI/CD

on:
  push:
    branches: [ main ]
    paths:
      - 'src/AIAspireSolution.AiService/**'
      - 'src/AIAspireSolution.Domain/**'
      - 'src/AIAspireSolution.ServiceDefaults/**'
  pull_request:
    branches: [ main ]
    paths:
      - 'src/AIAspireSolution.AiService/**'
      - 'src/AIAspireSolution.Domain/**'
      - 'src/AIAspireSolution.ServiceDefaults/**'
  workflow_dispatch:

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '9.0.x'
      
      - name: Restore dependencies
        run: dotnet restore AIAspireSolution.sln
      
      - name: Build
        run: dotnet build AIAspireSolution.sln --configuration Release --no-restore
      
      - name: Test
        run: dotnet test AIAspireSolution.sln --configuration Release --no-build
        env:
          AI__OpenAI__ApiKey: ${{ secrets.OPENAI_API_KEY }}
      
      - name: Publish artifacts
        if: github.event_name != 'pull_request'
        run: dotnet publish src/AIAspireSolution.AiService/AIAspireSolution.AiService.csproj -c Release -o publish
      
      - name: Upload artifacts
        if: github.event_name != 'pull_request'
        uses: actions/upload-artifact@v3
        with:
          name: ai-service
          path: publish
  
  deploy-to-azure:
    needs: build-and-test
    if: github.event_name != 'pull_request' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    
    steps:
      - name: Download artifacts
        uses: actions/download-artifact@v3
        with:
          name: ai-service
          path: publish
      
      - name: Login to Azure
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Deploy to Azure Container Apps
        uses: azure/container-apps-deploy-action@v1
        with:
          appSourcePath: ${{ github.workspace }}/publish
          acrName: ${{ secrets.ACR_NAME }}
          acrUsername: ${{ secrets.ACR_USERNAME }}
          acrPassword: ${{ secrets.ACR_PASSWORD }}
          containerAppName: ai-service
          resourceGroup: ${{ secrets.RESOURCE_GROUP }}
          environmentVariables: "AI__OpenAI__ApiKey=${{ secrets.OPENAI_API_KEY }}"
```

### Kubernetes Deployment

```yaml
# kubernetes/ai-service.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: ai-service
  namespace: ai-aspire
  labels:
    app: ai-service
    component: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ai-service
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: ai-service
        component: backend
    spec:
      containers:
        - name: ai-service
          image: aiaspiresolution.azurecr.io/ai-service:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 80
              name: http
            - containerPort: 8080
              name: grpc
          env:
            - name: ASPNETCORE_ENVIRONMENT
              value: Production
            - name: AI__OpenAI__ApiKey
              valueFrom:
                secretKeyRef:
                  name: ai-secrets
                  key: openai-api-key
            - name: ConnectionStrings__db
              valueFrom:
                secretKeyRef:
                  name: db-secrets
                  key: connection-string
          resources:
            requests:
              memory: "256Mi"
              cpu: "100m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /health/live
              port: 80
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health/ready
              port: 80
            initialDelaySeconds: 15
            periodSeconds: 10
      imagePullSecrets:
        - name: acr-auth
---
apiVersion: v1
kind: Service
metadata:
  name: ai-service
  namespace: ai-aspire
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      name: http
    - port: 8080
      targetPort: 8080
      protocol: TCP
      name: grpc
  selector:
    app: ai-service
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ai-service-ingress
  namespace: ai-aspire
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  tls:
  - hosts:
    - ai-api.example.com
    secretName: ai-api-tls
  rules:
  - host: ai-api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: ai-service
            port:
              number: 80
```

## Performance Optimization

### Caching Layer

```csharp
// AIAspireSolution.AiService/Services/CachingPromptService.cs
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using AIAspireSolution.Domain.Interfaces;
using AIAspireSolution.Domain.Models;
using Microsoft.Extensions.Caching.Distributed;
using Microsoft.Extensions.Logging;
using Microsoft.SemanticKernel;
using System.Text.Json;

namespace AIAspireSolution.AiService.Services;

/// <summary>
/// Caching decorator for PromptService that adds distributed caching
/// </summary>
public class CachingPromptService : IPromptService
{
    /// <summary>
    /// Decorated prompt service
    /// </summary>
    private readonly IPromptService _promptService;
    
    /// <summary>
    /// Distributed cache for prompt data
    /// </summary>
    private readonly IDistributedCache _cache;
    
    /// <summary>
    /// Logger for cache operations
    /// </summary>
    private readonly ILogger<CachingPromptService> _logger;
    
    /// <summary>
    /// Cache expiration options
    /// </summary>
    private readonly DistributedCacheEntryOptions _cacheOptions;

    /// <summary>
    /// Creates a new caching prompt service
    /// </summary>
    /// <param name="promptService">Prompt service to decorate</param>
    /// <param name="cache">Distributed cache</param>
    /// <param name="logger">Logger</param>
    public CachingPromptService(
        IPromptService promptService,
        IDistributedCache cache,
        ILogger<CachingPromptService> logger)
    {
        _promptService = promptService;
        _cache = cache;
        _logger = logger;
        
        // Set cache options - 1 hour expiration
        _cacheOptions = new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(1)
        };
    }

    /// <summary>
    /// Gets a prompt by ID with caching
    /// </summary>
    /// <param name="id">Prompt ID</param>
    /// <returns>Prompt or null</returns>
    public async Task<Prompt?> GetPromptAsync(Guid id)
    {
        var cacheKey = $"prompt:{id}";
        
        // Try to get from cache first
        var cachedPrompt = await _cache.GetStringAsync(cacheKey);
        if (cachedPrompt != null)
        {
            _logger.LogDebug("Cache hit for prompt ID: {PromptId}", id);
            return JsonSerializer.Deserialize<Prompt>(cachedPrompt);
        }
        
        // Cache miss, get from service
        var prompt = await _promptService.GetPromptAsync(id);
        if (prompt != null)
        {
            // Add to cache
            await _cache.SetStringAsync(
                cacheKey, 
                JsonSerializer.Serialize(prompt),
                _cacheOptions
            );
        }
        
        return prompt;
    }

    /// <summary>
    /// Gets a prompt by name with caching
    /// </summary>
    /// <param name="name">Prompt name</param>
    /// <returns>Prompt or null</returns>
    public async Task<Prompt?> GetPromptByNameAsync(string name)
    {
        var cacheKey = $"prompt:name:{name}";
        
        // Try to get from cache first
        var cachedPrompt = await _cache.GetStringAsync(cacheKey);
        if (cachedPrompt != null)
        {
            _logger.LogDebug("Cache hit for prompt name: {PromptName}", name);
            return JsonSerializer.Deserialize<Prompt>(cachedPrompt);
        }
        
        // Cache miss, get from service
        var prompt = await _promptService.GetPromptByNameAsync(name);
        if (prompt != null)
        {
            // Add to cache
            await _cache.SetStringAsync(
                cacheKey,
                JsonSerializer.Serialize(prompt),
                _cacheOptions
            );
            
            // Also cache by ID
            await _cache.SetStringAsync(
                $"prompt:{prompt.Id}",
                JsonSerializer.Serialize(prompt),
                _cacheOptions
            );
        }
        
        return prompt;
    }

    /// <summary>
    /// Invalidates the cache for a prompt
    /// </summary>
    /// <param name="prompt">Prompt to invalidate</param>
    private async Task InvalidateCacheAsync(Prompt prompt)
    {
        await _cache.RemoveAsync($"prompt:{prompt.Id}");
        await _cache.RemoveAsync($"prompt:name:{prompt.Name}");
        _logger.LogDebug("Cache invalidated for prompt: {PromptName} ({PromptId})",
            prompt.Name, prompt.Id);
    }

    // Forward remaining methods to the decorated service
    // and invalidate cache as needed

    public async Task<IEnumerable<Prompt>> GetAllPromptsAsync()
    {
        return await _promptService.GetAllPromptsAsync();
    }

    public async Task<IEnumerable<Prompt>> GetPromptsByCategoryAsync(string category)
    {
        return await _promptService.GetPromptsByCategoryAsync(category);
    }

    public async Task<IEnumerable<Prompt>> GetPromptsByTagsAsync(string[] tags)
    {
        return await _promptService.GetPromptsByTagsAsync(tags);
    }

    public async Task<Prompt> CreatePromptAsync(Prompt prompt)
    {
        var result = await _promptService.CreatePromptAsync(prompt);
        return result;
    }

    public async Task<Prompt> UpdatePromptAsync(Prompt prompt)
    {
        var result = await _promptService.UpdatePromptAsync(prompt);
        await InvalidateCacheAsync(result);
        return result;
    }

    public async Task<bool> DeletePromptAsync(Guid id)
    {
        var prompt = await GetPromptAsync(id);
        if (prompt != null)
        {
            var result = await _promptService.DeletePromptAsync(id);
            if (result)
            {
                await InvalidateCacheAsync(prompt);
            }
            return result;
        }
        return false;
    }

    public async Task<string> ProcessPromptTemplateAsync(string promptName, KernelArguments arguments)
    {
        // Process templates without caching - results are typically unique
        return await _promptService.ProcessPromptTemplateAsync(promptName, arguments);
    }

    public async Task<string> ProcessPromptTemplateByIdAsync(Guid promptId, KernelArguments arguments)
    {
        // Process templates without caching - results are typically unique
        return await _promptService.ProcessPromptTemplateByIdAsync(promptId, arguments);
    }
}
```

### Performance Monitoring

```csharp
// AIAspireSolution.ServiceDefaults/Extensions/TelemetryExtensions.cs
using System;
using System.Diagnostics;
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using OpenTelemetry.Metrics;
using OpenTelemetry.Resources;
using OpenTelemetry.Trace;

namespace AIAspireSolution.ServiceDefaults.Extensions;

/// <summary>
/// Extensions for configuring telemetry in services
/// </summary>
public static class TelemetryExtensions
{
    /// <summary>
    /// Configures comprehensive observability for AI services
    /// </summary>
    /// <param name="builder">Web application builder</param>
    /// <returns>Builder for chaining</returns>
    public static WebApplicationBuilder AddAIObservability(this WebApplicationBuilder builder)
    {
        // Create resource attributes
        var resourceBuilder = ResourceBuilder.CreateDefault()
            .AddService(builder.Environment.ApplicationName)
            .AddAttributes(new KeyValuePair<string, object>[]
            {
                new("application.component", "ai-service"),
                new("deployment.environment", builder.Environment.EnvironmentName)
            });
        
        // Configure OpenTelemetry
        builder.Services.AddOpenTelemetry()
            // Configure metrics
            .WithMetrics(metrics =>
            {
                metrics
                    .SetResourceBuilder(resourceBuilder)
                    .AddRuntimeInstrumentation()
                    .AddAspNetCoreInstrumentation()
                    .AddHttpClientInstrumentation()
                    .AddProcessInstrumentation()
                    // Add AI-specific meters
                    .AddMeter("AIAspireSolution.AiService")
                    .AddOtlpExporter();
            })
            // Configure tracing
            .WithTracing(tracing =>
            {
                tracing
                    .SetResourceBuilder(resourceBuilder)
                    .AddAspNetCoreInstrumentation()
                    .AddHttpClientInstrumentation()
                    .AddEntityFrameworkCoreInstrumentation()
                    .AddGrpcClientInstrumentation()
                    .AddSource("AIAspireSolution.AiService")
                    .AddO

---


# Comprehensive Technical Documentation: Building AI-Powered Applications with .NET Aspire and Semantic Kernel (continued)

**Current Date: 2025-03-17 20:04:03 UTC**

## Performance Monitoring (continued)

```csharp
// AIAspireSolution.ServiceDefaults/Extensions/TelemetryExtensions.cs (continued)
            .WithTracing(tracing =>
            {
                tracing
                    .SetResourceBuilder(resourceBuilder)
                    .AddAspNetCoreInstrumentation()
                    .AddHttpClientInstrumentation()
                    .AddEntityFrameworkCoreInstrumentation()
                    .AddGrpcClientInstrumentation()
                    .AddSource("AIAspireSolution.AiService")
                    .AddOtlpExporter();
            });
            
        // Add AI-specific metrics
        builder.Services.AddSingleton<AIMetrics>();
        
        return builder;
    }
}
```

```csharp
// AIAspireSolution.ServiceDefaults/Telemetry/AIMetrics.cs
using System;
using System.Diagnostics;
using System.Diagnostics.Metrics;

namespace AIAspireSolution.ServiceDefaults.Telemetry;

/// <summary>
/// Custom metrics for AI operations
/// </summary>
public class AIMetrics
{
    /// <summary>
    /// Meter for recording AI metrics
    /// </summary>
    private readonly Meter _meter;
    
    /// <summary>
    /// Counter for AI completions
    /// </summary>
    public readonly Counter<long> CompletionsRequested;
    
    /// <summary>
    /// Histogram for AI completion tokens used
    /// </summary>
    public readonly Histogram<int> CompletionTokensUsed;
    
    /// <summary>
    /// Histogram for AI completion latency
    /// </summary>
    public readonly Histogram<double> CompletionLatencyMs;
    
    /// <summary>
    /// Counter for AI errors
    /// </summary>
    public readonly Counter<long> AIErrors;
    
    /// <summary>
    /// Creates a new metrics recorder for AI operations
    /// </summary>
    public AIMetrics()
    {
        _meter = new Meter("AIAspireSolution.AiService", "1.0.0");
        
        CompletionsRequested = _meter.CreateCounter<long>(
            name: "ai.completions.count",
            unit: "{completions}",
            description: "Number of AI completions requested");
            
        CompletionTokensUsed = _meter.CreateHistogram<int>(
            name: "ai.completions.tokens",
            unit: "{tokens}",
            description: "Number of tokens used per completion");
            
        CompletionLatencyMs = _meter.CreateHistogram<double>(
            name: "ai.completions.latency",
            unit: "ms",
            description: "Latency of AI completions in milliseconds");
            
        AIErrors = _meter.CreateCounter<long>(
            name: "ai.errors.count",
            unit: "{errors}",
            description: "Number of AI errors encountered");
    }
    
    /// <summary>
    /// Records metrics for a completed AI operation
    /// </summary>
    /// <param name="modelId">ID of the model used</param>
    /// <param name="tokensUsed">Number of tokens used</param>
    /// <param name="latencyMs">Latency in milliseconds</param>
    /// <param name="successful">Whether the operation was successful</param>
    public void RecordCompletion(string modelId, int tokensUsed, double latencyMs, bool successful)
    {
        var tags = new TagList
        {
            { "model", modelId },
            { "success", successful.ToString() }
        };
        
        CompletionsRequested.Add(1, tags);
        
        if (successful)
        {
            CompletionTokensUsed.Record(tokensUsed, tags);
            CompletionLatencyMs.Record(latencyMs, tags);
        }
        else
        {
            AIErrors.Add(1, tags);
        }
    }
}
```

### Performance Optimization Middleware

```csharp
// AIAspireSolution.AiService/Middleware/AIPerformanceMiddleware.cs
using System;
using System.Diagnostics;
using System.Threading.Tasks;
using AIAspireSolution.ServiceDefaults.Telemetry;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;

namespace AIAspireSolution.AiService.Middleware;

/// <summary>
/// Middleware that tracks performance metrics for AI operations
/// </summary>
public class AIPerformanceMiddleware
{
    /// <summary>
    /// Next middleware in the pipeline
    /// </summary>
    private readonly RequestDelegate _next;
    
    /// <summary>
    /// Logger for middleware operations
    /// </summary>
    private readonly ILogger<AIPerformanceMiddleware> _logger;
    
    /// <summary>
    /// Metrics recorder
    /// </summary>
    private readonly AIMetrics _metrics;
    
    /// <summary>
    /// Creates a new performance middleware
    /// </summary>
    /// <param name="next">Next middleware</param>
    /// <param name="logger">Logger</param>
    /// <param name="metrics">Metrics recorder</param>
    public AIPerformanceMiddleware(
        RequestDelegate next,
        ILogger<AIPerformanceMiddleware> logger,
        AIMetrics metrics)
    {
        _next = next;
        _logger = logger;
        _metrics = metrics;
    }
    
    /// <summary>
    /// Processes the request and records performance metrics
    /// </summary>
    /// <param name="context">HTTP context</param>
    public async Task InvokeAsync(HttpContext context)
    {
        // Skip non-AI endpoints
        if (!IsAIEndpoint(context.Request.Path))
        {
            await _next(context);
            return;
        }
        
        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            // Process the request
            await _next(context);
            
            stopwatch.Stop();
            
            // Record metrics for successful requests
            if (context.Response.StatusCode < 400)
            {
                // Extract model and token information from response headers
                if (context.Response.Headers.TryGetValue("X-AI-Model", out var model) &&
                    context.Response.Headers.TryGetValue("X-AI-Tokens", out var tokens) &&
                    int.TryParse(tokens, out var tokenCount))
                {
                    _metrics.RecordCompletion(
                        model.ToString(),
                        tokenCount,
                        stopwatch.Elapsed.TotalMilliseconds,
                        true);
                        
                    _logger.LogDebug("AI operation completed: {Model}, {Tokens} tokens, {LatencyMs}ms",
                        model, tokenCount, stopwatch.Elapsed.TotalMilliseconds);
                }
            }
            else
            {
                // Record error metrics
                _metrics.RecordCompletion(
                    "unknown", // model might not be available for errors
                    0,
                    stopwatch.Elapsed.TotalMilliseconds,
                    false);
                    
                _logger.LogWarning("AI operation failed: Status {StatusCode}, {LatencyMs}ms",
                    context.Response.StatusCode, stopwatch.Elapsed.TotalMilliseconds);
            }
        }
        catch (Exception ex)
        {
            stopwatch.Stop();
            
            // Record exception metrics
            _metrics.RecordCompletion(
                "unknown",
                0,
                stopwatch.Elapsed.TotalMilliseconds,
                false);
                
            _logger.LogError(ex, "Exception in AI operation: {LatencyMs}ms",
                stopwatch.Elapsed.TotalMilliseconds);
                
            throw;
        }
    }
    
    /// <summary>
    /// Determines if a path is for an AI endpoint
    /// </summary>
    /// <param name="path">Request path</param>
    /// <returns>True if this is an AI endpoint</returns>
    private bool IsAIEndpoint(PathString path)
    {
        return path.StartsWithSegments("/api/ai") || 
               path.StartsWithSegments("/v1/ai") ||
               path.Value?.Contains("/ai/") == true;
    }
}
```

## Security Best Practices

### Authentication Configuration

```csharp
// AIAspireSolution.AiService/Program.cs
// Add authentication configuration

// Add authentication
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        // Get configuration
        var config = builder.Configuration;
        
        options.Authority = config["Auth:Authority"];
        options.Audience = config["Auth:Audience"];
        options.RequireHttpsMetadata = !builder.Environment.IsDevelopment();
        
        // Handle token validation events
        options.Events = new JwtBearerEvents
        {
            OnAuthenticationFailed = context =>
            {
                var logger = context.HttpContext.RequestServices
                    .GetRequiredService<ILogger<Program>>();
                    
                logger.LogError(context.Exception, "Authentication failed");
                return Task.CompletedTask;
            },
            OnTokenValidated = context =>
            {
                var tokenValidatorService = context.HttpContext.RequestServices
                    .GetRequiredService<ITokenValidatorService>();
                    
                return tokenValidatorService.ValidateTokenAsync(context);
            }
        };
    });

// Add authorization with policies
builder.Services.AddAuthorization(options =>
{
    // Basic access policy
    options.AddPolicy("AIServiceAccess", policy =>
        policy.RequireAuthenticatedUser()
              .RequireClaim("scope", "ai-service:access"));
              
    // Admin policy
    options.AddPolicy("AIServiceAdmin", policy =>
        policy.RequireAuthenticatedUser()
              .RequireClaim("scope", "ai-service:admin")
              .RequireRole("Admin"));
              
    // Token budget policy
    options.AddPolicy("AITokenBudget", policy =>
        policy.Requirements.Add(new AITokenBudgetRequirement()));
});

// Register authorization handlers
builder.Services.AddTransient<IAuthorizationHandler, AITokenBudgetHandler>();
```

### Token Budget Authorization Handler

```csharp
// AIAspireSolution.AiService/Security/AITokenBudgetHandler.cs
using System;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authorization;
using Microsoft.Extensions.Logging;
using AIAspireSolution.Domain.Interfaces;

namespace AIAspireSolution.AiService.Security;

/// <summary>
/// Authorization requirement for checking AI token budgets
/// </summary>
public class AITokenBudgetRequirement : IAuthorizationRequirement
{
    /// <summary>
    /// Minimum token budget required for authorization
    /// </summary>
    public int MinimumTokenBudget { get; } = 1000;
}

/// <summary>
/// Handler for checking AI token budgets
/// </summary>
public class AITokenBudgetHandler : AuthorizationHandler<AITokenBudgetRequirement>
{
    /// <summary>
    /// Budget service for checking user token allocations
    /// </summary>
    private readonly ITokenBudgetService _budgetService;
    
    /// <summary>
    /// Logger for handler operations
    /// </summary>
    private readonly ILogger<AITokenBudgetHandler> _logger;
    
    /// <summary>
    /// Creates a new budget handler
    /// </summary>
    /// <param name="budgetService">Budget service</param>
    /// <param name="logger">Logger</param>
    public AITokenBudgetHandler(ITokenBudgetService budgetService, ILogger<AITokenBudgetHandler> logger)
    {
        _budgetService = budgetService;
        _logger = logger;
    }
    
    /// <summary>
    /// Handles the authorization requirement
    /// </summary>
    /// <param name="context">Authorization context</param>
    /// <param name="requirement">Budget requirement</param>
    protected override async Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        AITokenBudgetRequirement requirement)
    {
        // Skip for anonymous users
        if (!context.User.Identity?.IsAuthenticated ?? true)
        {
            _logger.LogDebug("Skipping token budget check for anonymous user");
            return;
        }
        
        // Get user ID from claims
        var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        if (string.IsNullOrEmpty(userId))
        {
            _logger.LogWarning("User ID claim not found in token");
            return;
        }
        
        try
        {
            // Check token budget
            var budget = await _budgetService.GetRemainingBudgetAsync(userId);
            
            if (budget >= requirement.MinimumTokenBudget)
            {
                _logger.LogDebug("Token budget check passed for user {UserId}: {RemainingBudget} tokens",
                    userId, budget);
                context.Succeed(requirement);
            }
            else
            {
                _logger.LogWarning("Token budget check failed for user {UserId}: only {RemainingBudget} tokens remaining",
                    userId, budget);
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error checking token budget for user {UserId}", userId);
        }
    }
}
```

### API Key Validation

```csharp
// AIAspireSolution.AiService/Security/ApiKeyAuthFilter.cs
using System;
using System.Linq;
using System.Security.Claims;
using System.Text.Encodings.Web;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Options;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Filters;
using AIAspireSolution.Domain.Interfaces;

namespace AIAspireSolution.AiService.Security;

/// <summary>
/// Authentication scheme handler for API keys
/// </summary>
public class ApiKeyAuthenticationHandler : AuthenticationHandler<AuthenticationSchemeOptions>
{
    /// <summary>
    /// API key validation service
    /// </summary>
    private readonly IApiKeyService _apiKeyService;
    
    /// <summary>
    /// Creates a new API key handler
    /// </summary>
    /// <param name="options">Authentication options</param>
    /// <param name="logger">Logger</param>
    /// <param name="encoder">URL encoder</param>
    /// <param name="clock">System clock</param>
    /// <param name="apiKeyService">API key service</param>
    public ApiKeyAuthenticationHandler(
        IOptionsMonitor<AuthenticationSchemeOptions> options,
        ILoggerFactory logger,
        UrlEncoder encoder,
        ISystemClock clock,
        IApiKeyService apiKeyService)
        : base(options, logger, encoder, clock)
    {
        _apiKeyService = apiKeyService;
    }
    
    /// <summary>
    /// Authenticates requests using API key
    /// </summary>
    /// <returns>Authentication result</returns>
    protected override async Task<AuthenticateResult> HandleAuthenticateAsync()
    {
        // Check header for API key
        if (!Request.Headers.TryGetValue("X-API-Key", out var apiKeyHeaderValues))
        {
            // API key not in header
            return AuthenticateResult.NoResult();
        }
        
        var apiKey = apiKeyHeaderValues.FirstOrDefault();
        if (string.IsNullOrEmpty(apiKey))
        {
            // Empty API key
            return AuthenticateResult.NoResult();
        }
        
        try
        {
            // Validate API key
            var apiKeyInfo = await _apiKeyService.ValidateApiKeyAsync(apiKey);
            if (apiKeyInfo == null)
            {
                Logger.LogWarning("Invalid API key attempted: {ApiKeyPrefix}",
                    apiKey.Length > 5 ? apiKey.Substring(0, 5) + "..." : apiKey);
                return AuthenticateResult.Fail("Invalid API key");
            }
            
            // Create claims principal for the authenticated client
            var claims = new[]
            {
                new Claim(ClaimTypes.Name, apiKeyInfo.ClientName),
                new Claim(ClaimTypes.NameIdentifier, apiKeyInfo.ClientId.ToString()),
                new Claim("scope", "ai-service:access")
                // Add any additional scopes from apiKeyInfo
            };
            
            var identity = new ClaimsIdentity(claims, Scheme.Name);
            var principal = new ClaimsPrincipal(identity);
            var ticket = new AuthenticationTicket(principal, Scheme.Name);
            
            Logger.LogInformation("Successful API key authentication for client {ClientName}",
                apiKeyInfo.ClientName);
                
            return AuthenticateResult.Success(ticket);
        }
        catch (Exception ex)
        {
            Logger.LogError(ex, "Exception during API key authentication");
            return AuthenticateResult.Fail("Authentication failed due to an error");
        }
    }
}
```

### Content Security and Sanitization

```csharp
// AIAspireSolution.AiService/Services/ContentSanitizationService.cs
using System;
using System.Collections.Generic;
using System.Text.RegularExpressions;
using System.Threading.Tasks;
using Ganss.Xss;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Options;

namespace AIAspireSolution.AiService.Services;

/// <summary>
/// Service for sanitizing and validating content
/// </summary>
public class ContentSanitizationService
{
    /// <summary>
    /// HTML sanitizer
    /// </summary>
    private readonly HtmlSanitizer _htmlSanitizer;
    
    /// <summary>
    /// Logger for sanitization operations
    /// </summary>
    private readonly ILogger<ContentSanitizationService> _logger;
    
    /// <summary>
    /// Patterns for detecting sensitive content
    /// </summary>
    private readonly List<Regex> _sensitivePatterns = new();
    
    /// <summary>
    /// Creates a new content sanitization service
    /// </summary>
    /// <param name="options">Configuration options</param>
    /// <param name="logger">Logger</param>
    public ContentSanitizationService(
        IOptions<ContentSecurityOptions> options,
        ILogger<ContentSanitizationService> logger)
    {
        _logger = logger;
        
        // Configure HTML sanitizer
        _htmlSanitizer = new HtmlSanitizer();
        _htmlSanitizer.AllowedTags.Clear();
        _htmlSanitizer.AllowedTags.Add("p");
        _htmlSanitizer.AllowedTags.Add("br");
        _htmlSanitizer.AllowedTags.Add("ul");
        _htmlSanitizer.AllowedTags.Add("ol");
        _htmlSanitizer.AllowedTags.Add("li");
        _htmlSanitizer.AllowedTags.Add("b");
        _htmlSanitizer.AllowedTags.Add("i");
        _htmlSanitizer.AllowedTags.Add("code");
        _htmlSanitizer.AllowedTags.Add("pre");
        
        // Add PII detection patterns
        var config = options.Value;
        
        // Email pattern
        _sensitivePatterns.Add(new Regex(config.EmailPattern ?? @"[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}", 
            RegexOptions.Compiled | RegexOptions.IgnoreCase));
            
        // Phone number pattern
        _sensitivePatterns.Add(new Regex(config.PhonePattern ?? @"\b(\+\d{1,3}[\s-]?)?\(?\d{3}\)?[\s.-]?\d{3}[\s.-]?\d{4}\b", 
            RegexOptions.Compiled));
            
        // Additional patterns from configuration
        foreach (var pattern in config.AdditionalPatterns)
        {
            _sensitivePatterns.Add(new Regex(pattern, RegexOptions.Compiled));
        }
    }
    
    /// <summary>
    /// Sanitizes HTML content
    /// </summary>
    /// <param name="html">HTML to sanitize</param>
    /// <returns>Sanitized HTML</returns>
    public string SanitizeHtml(string html)
    {
        if (string.IsNullOrEmpty(html))
        {
            return html;
        }
        
        var sanitized = _htmlSanitizer.Sanitize(html);
        
        // If content was modified, log it
        if (sanitized != html)
        {
            _logger.LogWarning("Content was sanitized due to disallowed HTML");
        }
        
        return sanitized;
    }
    
    /// <summary>
    /// Checks content for potentially sensitive information
    /// </summary>
    /// <param name="content">Content to check</param>
    /// <returns>Result with detected sensitive information</returns>
    public ContentScanResult ScanForSensitiveContent(string content)
    {
        var result = new ContentScanResult
        {
            ContainsSensitiveContent = false
        };
        
        if (string.IsNullOrEmpty(content))
        {
            return result;
        }
        
        foreach (var pattern in _sensitivePatterns)
        {
            var matches = pattern.Matches(content);
            if (matches.Count > 0)
            {
                result.ContainsSensitiveContent = true;
                
                foreach (Match match in matches)
                {
                    result.DetectedPatterns.Add(new DetectedPattern
                    {
                        Pattern = pattern.ToString(),
                        Value = match.Value,
                        Position = match.Index
                    });
                }
            }
        }
        
        if (result.ContainsSensitiveContent)
        {
            _logger.LogWarning("Detected {Count} instances of potentially sensitive content",
                result.DetectedPatterns.Count);
        }
        
        return result;
    }
    
    /// <summary>
    /// Redacts sensitive information from content
    /// </summary>
    /// <param name="content">Content to redact</param>
    /// <returns>Redacted content</returns>
    public string RedactSensitiveContent(string content)
    {
        if (string.IsNullOrEmpty(content))
        {
            return content;
        }
        
        var result = content;
        
        foreach (var pattern in _sensitivePatterns)
        {
            // Replace with appropriate redaction
            result = pattern.Replace(result, "[REDACTED]");
        }
        
        return result;
    }
}

/// <summary>
/// Result of sensitive content scan
/// </summary>
public class ContentScanResult
{
    /// <summary>
    /// Whether sensitive content was detected
    /// </summary>
    public bool ContainsSensitiveContent { get; set; }
    
    /// <summary>
    /// List of detected patterns
    /// </summary>
    public List<DetectedPattern> DetectedPatterns { get; set; } = new();
}

/// <summary>
/// Information about a detected pattern
/// </summary>
public class DetectedPattern
{
    /// <summary>
    /// Pattern that was matched
    /// </summary>
    public string Pattern { get; set; } = string.Empty;
    
    /// <summary>
    /// Value that matched the pattern
    /// </summary>
    public string Value { get; set; } = string.Empty;
    
    /// <summary>
    /// Position in the content where the pattern was found
    /// </summary>
    public int Position { get; set; }
}

/// <summary>
/// Configuration options for content security
/// </summary>
public class ContentSecurityOptions
{
    /// <summary>
    /// Regular expression pattern for detecting emails
    /// </summary>
    public string? EmailPattern { get; set; }
    
    /// <summary>
    /// Regular expression pattern for detecting phone numbers
    /// </summary>
    public string? PhonePattern { get; set; }
    
    /// <summary>
    /// Additional patterns to check for
    /// </summary>
    public List<string> AdditionalPatterns { get; set; } = new();
    
    /// <summary>
    /// Maximum content length to allow
    /// </summary>
    public int MaxContentLength { get; set; } = 10000;
}
```

## Monitoring and Observability

### Health Checks

```csharp
// AIAspireSolution.AiService/HealthChecks/AIServiceHealthCheck.cs
using System;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Extensions.Diagnostics.HealthChecks;
using Microsoft.Extensions.Logging;
using Microsoft.SemanticKernel;

namespace AIAspireSolution.AiService.HealthChecks;

/// <summary>
/// Health check that verifies the AI service is working
/// </summary>
public class AIServiceHealthCheck : IHealthCheck
{
    /// <summary>
    /// Semantic Kernel instance
    /// </summary>
    private readonly Kernel _kernel;
    
    /// <summary>
    /// Logger for health check operations
    /// </summary>
    private readonly ILogger<AIServiceHealthCheck> _logger;
    
    /// <summary>
    /// Creates a new AI service health check
    /// </summary>
    /// <param name="kernel">Semantic Kernel</param>
    /// <param name="logger">Logger</param>
    public AIServiceHealthCheck(Kernel kernel, ILogger<AIServiceHealthCheck> logger)
    {
        _kernel = kernel;
        _logger = logger;
    }
    
    /// <summary>
    /// Performs health check by testing the AI service
    /// </summary>
    /// <param name="context">Health check context</param>
    /// <param name="cancellationToken">Cancellation token</param>
    /// <returns>Health check result</returns>
    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context, 
        CancellationToken cancellationToken = default)
    {
        try
        {
            // Create a simple test prompt
            var prompt = "Return the text 'HEALTHY' without any additional text.";
            var function = _kernel.CreateFunctionFromPrompt(prompt);
            
            // Execute with timeout
            var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
            var linkedCts = CancellationTokenSource.CreateLinkedTokenSource(
                cancellationToken, cts.Token);
                
            var result = await _kernel.InvokeAsync(function, cancellationToken: linkedCts.Token);
            var response = result.GetValue<string>();
            
            // Check if response contains "HEALTHY"
            if (!string.IsNullOrEmpty(response) && response.Contains("HEALTHY"))
            {
                return HealthCheckResult.Healthy("AI service is responding correctly");
            }
            
            // If response doesn't match expected output
            _logger.LogWarning("AI service health check response doesn't match expected output: {Response}",
                response);
                
            return HealthCheckResult.Degraded("AI service response: " + response);
        }
        catch (OperationCanceledException)
        {
            _logger.LogWarning("AI service health check timed out after 5 seconds");
            return HealthCheckResult.Degraded("AI service response timed out");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "AI service health check failed");
            return HealthCheckResult.Unhealthy("AI service error: " + ex.Message);
        }
    }
}
```

### Grafana Dashboard Configuration

```json
{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "datasource": "-- Grafana --",
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "type": "dashboard"
      },
      {
        "datasource": "Loki",
        "enable": true,
        "expr": "{app=\"ai-service\"} |= \"error\"",
        "iconColor": "rgba(255, 96, 96, 1)",
        "name": "AI Service Errors",
        "titleFormat": "Error detected"
      }
    ]
  },
  "editable": true,
  "gnetId": null,
  "graphTooltip": 0,
  "id": 12,
  "links": [],
  "panels": [
    {
      "datasource": null,
      "gridPos": {
        "h": 2,
        "w": 24,
        "x": 0,
        "y": 0
      },
      "id": 1,
      "title": "AI Service Dashboard",
      "type": "text",
      "content": "# AI Service Monitoring\nThis dashboard shows real-time metrics for the AI service including completions, latency, and errors."
    },
    {
      "datasource": "Prometheus",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        }
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 0,
        "y": 2
      },
      "id": 2,
      "options": {
        "stat": {
          "colorMode": "value",
          "noValue": "0",
          "unit": "none"
        },
        "text": {
          "titleSize": 32,
          "valueSize": 24
        }
      },
      "pluginVersion": "8.0.6",
      "title": "Total Completions",
      "type": "stat",
      "targets": [
        {
          "expr": "sum(increase(ai_completions_count_total{job=\"ai-service\"}[$__interval]))",
          "interval": "",
          "legendFormat": "",
          "refId": "A"
        }
      ]
    },
    {
      "datasource": "Prometheus",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 1000
              },
              {
                "color": "red",
                "value": 2000
              }
            ]
          },
          "unit": "ms"
        }
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 12,
        "y": 2
      },
      "id": 3,
      "options": {
        "orientation
---

# Comprehensive Technical Documentation: Building AI-Powered Applications with .NET Aspire and Semantic Kernel (continued)

**Current Date: 2025-03-17 20:13:22 UTC**

## Monitoring and Observability (continued)

### Grafana Dashboard Configuration (continued)

```json
      "options": {
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "mean"
          ],
          "fields": "",
          "values": false
        },
        "showThresholdLabels": false,
        "showThresholdMarkers": true,
        "text": {}
      },
      "pluginVersion": "8.0.6",
      "title": "Average Latency",
      "type": "gauge",
      "targets": [
        {
          "expr": "histogram_quantile(0.95, sum(rate(ai_completions_latency_bucket{job=\"ai-service\"}[$__interval])) by (le))",
          "interval": "",
          "legendFormat": "p95",
          "refId": "A"
        }
      ]
    },
    {
      "datasource": "Prometheus",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 20,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "smooth",
            "lineWidth": 2,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "never",
            "spanNulls": true,
            "stacking": {
              "group": "A",
              "mode": "none"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          },
          "unit": "short"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 0,
        "y": 10
      },
      "id": 4,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom"
        },
        "tooltip": {
          "mode": "single"
        }
      },
      "pluginVersion": "8.0.6",
      "title": "Completions Over Time",
      "type": "timeseries",
      "targets": [
        {
          "expr": "sum(increase(ai_completions_count_total{job=\"ai-service\"}[$__interval])) by (model)",
          "interval": "",
          "legendFormat": "{{model}}",
          "refId": "A"
        }
      ]
    },
    {
      "datasource": "Prometheus",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 20,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "smooth",
            "lineWidth": 2,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "never",
            "spanNulls": true,
            "stacking": {
              "group": "A",
              "mode": "none"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 5
              }
            ]
          },
          "unit": "short"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 12,
        "y": 10
      },
      "id": 5,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom"
        },
        "tooltip": {
          "mode": "single"
        }
      },
      "pluginVersion": "8.0.6",
      "title": "Error Rate",
      "type": "timeseries",
      "targets": [
        {
          "expr": "sum(increase(ai_errors_count_total{job=\"ai-service\"}[$__interval]))",
          "interval": "",
          "legendFormat": "Errors",
          "refId": "A"
        }
      ]
    },
    {
      "datasource": "Prometheus",
      "description": "Token usage by model",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            }
          },
          "mappings": []
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 0,
        "y": 18
      },
      "id": 6,
      "options": {
        "displayLabels": ["name", "percent"],
        "legend": {
          "displayMode": "table",
          "placement": "right",
          "values": ["value", "percent"]
        },
        "pieType": "donut",
        "reduceOptions": {
          "calcs": ["sum"],
          "fields": "",
          "values": false
        },
        "tooltip": {
          "mode": "single"
        }
      },
      "pluginVersion": "8.0.6",
      "title": "Token Usage by Model",
      "type": "piechart",
      "targets": [
        {
          "expr": "sum(ai_completions_tokens_sum{job=\"ai-service\"}) by (model)",
          "interval": "",
          "legendFormat": "{{model}}",
          "refId": "A"
        }
      ]
    },
    {
      "datasource": "Loki",
      "description": "Recent errors from AI service logs",
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 12,
        "y": 18
      },
      "id": 7,
      "options": {
        "dedupStrategy": "none",
        "enableLogDetails": true,
        "prettifyLogMessage": true,
        "showCommonLabels": false,
        "showLabels": false,
        "showTime": true,
        "sortOrder": "Descending",
        "wrapLogMessage": true
      },
      "pluginVersion": "8.0.6",
      "targets": [
        {
          "expr": "{app=\"ai-service\"} |= \"error\" | json",
          "refId": "A"
        }
      ],
      "title": "Recent Errors",
      "type": "logs"
    }
  ],
  "refresh": "30s",
  "schemaVersion": 30,
  "style": "dark",
  "tags": ["ai-service", "semantic-kernel"],
  "templating": {
    "list": [
      {
        "allValue": null,
        "current": {
          "selected": false,
          "text": "All",
          "value": "$__all"
        },
        "datasource": "Prometheus",
        "definition": "label_values(ai_completions_count_total, model)",
        "description": "AI Model",
        "error": null,
        "hide": 0,
        "includeAll": true,
        "label": "Model",
        "multi": false,
        "name": "model",
        "options": [],
        "query": {
          "query": "label_values(ai_completions_count_total, model)",
          "refId": "StandardVariableQuery"
        },
        "refresh": 1,
        "regex": "",
        "skipUrlSync": false,
        "sort": 1,
        "type": "query"
      }
    ]
  },
  "time": {
    "from": "now-6h",
    "to": "now"
  },
  "timepicker": {
    "refresh_intervals": [
      "5s",
      "10s",
      "30s",
      "1m",
      "5m",
      "15m",
      "30m",
      "1h",
      "2h",
      "1d"
    ]
  },
  "timezone": "browser",
  "title": "AI Service Dashboard",
  "uid": "ai-service-dashboard",
  "version": 1
}
```

### Logging Configuration

```csharp
// AIAspireSolution.ServiceDefaults/Extensions/LoggingExtensions.cs
using System;
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Serilog;
using Serilog.Enrichers.Span;
using Serilog.Events;
using Serilog.Exceptions;
using Serilog.Sinks.OpenTelemetry;

namespace AIAspireSolution.ServiceDefaults.Extensions;

/// <summary>
/// Extensions for configuring structured logging
/// </summary>
public static class LoggingExtensions
{
    /// <summary>
    /// Adds structured logging with Serilog
    /// </summary>
    /// <param name="builder">Web application builder</param>
    /// <returns>Builder for chaining</returns>
    public static WebApplicationBuilder AddStructuredLogging(this WebApplicationBuilder builder)
    {
        // Create Serilog logger configuration
        Log.Logger = new LoggerConfiguration()
            // Minimum level based on environment
            .MinimumLevel.Information()
            // Override levels for specific categories
            .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
            .MinimumLevel.Override("Microsoft.AspNetCore", LogEventLevel.Warning)
            // Enrich logs with contextual information
            .Enrichers.WithSpan()
            .Enrichers.WithMachineName()
            .Enrichers.WithEnvironmentName()
            .Enrichers.WithExceptionDetails()
            // Add contextual properties from LogContext
            .Enrich.FromLogContext()
            // Configure sinks - console for local viewing
            .WriteTo.Console(
                outputTemplate: "[{Timestamp:yyyy-MM-dd HH:mm:ss.fff} {Level:u3}] {Message:lj} " +
                                "{Properties:j}{NewLine}{Exception}")
            // OpenTelemetry for centralized logging
            .WriteTo.OpenTelemetry(options =>
            {
                options.Endpoint = builder.Configuration["Telemetry:OtlpEndpoint"];
                options.Protocol = builder.Configuration["Telemetry:OtlpProtocol"] == "http" ?
                    OtlpProtocol.HttpProtobuf : OtlpProtocol.Grpc;
                options.ResourceAttributes = new[]
                {
                    new KeyValuePair<string, object>("service.name", builder.Environment.ApplicationName),
                    new KeyValuePair<string, object>("deployment.environment", builder.Environment.EnvironmentName)
                };
            })
            .CreateLogger();
            
        // Clear default logging providers
        builder.Logging.ClearProviders();
        
        // Add Serilog
        builder.Host.UseSerilog();
        
        return builder;
    }
    
    /// <summary>
    /// Configures AI-specific log enrichment
    /// </summary>
    /// <param name="app">Web application</param>
    /// <returns>Application for chaining</returns>
    public static WebApplication UseAILogEnrichment(this WebApplication app)
    {
        // Add middleware to enrich logs with AI-specific data
        app.Use(async (context, next) => 
        {
            // Add request trace identifier
            using (Serilog.Context.LogContext.PushProperty("RequestId", context.TraceIdentifier))
            // Add user identifier if authenticated
            using (context.User?.Identity?.IsAuthenticated == true
                ? Serilog.Context.LogContext.PushProperty("UserId", 
                    context.User.FindFirst("sub")?.Value ?? "unknown")
                : null)
            {
                // Extract AI model from headers if present
                if (context.Request.Headers.TryGetValue("X-AI-Model", out var model))
                {
                    using (Serilog.Context.LogContext.PushProperty("AIModel", model.ToString()))
                    {
                        await next();
                    }
                }
                else
                {
                    await next();
                }
            }
        });
        
        return app;
    }
}
```

### Trace Sampling Configuration

```csharp
// AIAspireSolution.ServiceDefaults/Extensions/TracingExtensions.cs
using System.Collections.Generic;
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using OpenTelemetry.Trace;
using OpenTelemetry.Resources;

namespace AIAspireSolution.ServiceDefaults.Extensions;

/// <summary>
/// Extensions for configuring distributed tracing
/// </summary>
public static class TracingExtensions
{
    /// <summary>
    /// Adds AI-specific trace configuration
    /// </summary>
    /// <param name="builder">Web application builder</param>
    /// <returns>Builder for chaining</returns>
    public static WebApplicationBuilder AddAITracing(this WebApplicationBuilder builder)
    {
        // Configure resource attributes for proper identification
        var resourceBuilder = ResourceBuilder.CreateDefault()
            .AddService(builder.Environment.ApplicationName)
            .AddAttributes(new KeyValuePair<string, object>[]
            {
                new("ai.version", "1.0.0"),
                new("application.component", "ai-service")
            });

        // Add OpenTelemetry with specialized trace sampling
        builder.Services.AddOpenTelemetry()
            .WithTracing(tracing =>
            {
                tracing
                    .SetResourceBuilder(resourceBuilder)
                    .AddSource("AIAspireSolution.AiService") // Our custom source
                    .AddAspNetCoreInstrumentation(opts =>
                    {
                        // Configure custom filter for sensitive endpoints
                        opts.Filter = ctx => 
                        {
                            // Don't trace health check endpoints
                            if (ctx.Request.Path.StartsWithSegments("/health"))
                            {
                                return false;
                            }
                            
                            // Always trace AI endpoints for debugging
                            if (ctx.Request.Path.StartsWithSegments("/api/ai"))
                            {
                                return true;
                            }
                            
                            return true;
                        };
                        
                        // Enrich spans with custom tags
                        opts.EnrichWithHttpRequest = (activity, request) =>
                        {
                            if (request.Headers.TryGetValue("X-API-Key", out var apiKey))
                            {
                                // Add API key identifier but not the actual key
                                activity.SetTag("ai.api_key_present", "true");
                                
                                // If key starts with test prefix, mark for debugging
                                if (apiKey.ToString().StartsWith("test_"))
                                {
                                    activity.SetTag("ai.test_request", "true");
                                }
                            }
                            
                            // Add AI model if present
                            if (request.Headers.TryGetValue("X-AI-Model", out var model))
                            {
                                activity.SetTag("ai.model", model.ToString());
                            }
                        };
                    })
                    .AddHttpClientInstrumentation()
                    .AddEntityFrameworkCoreInstrumentation()
                    .AddGrpcClientInstrumentation()
                    // Configure sampling for efficiency
                    .SetSampler(new CompositeTracer(
                        // Sample all AI endpoints
                        new CustomAIEndpointSampler(),
                        // Use probabilistic sampling for other traffic
                        new TraceIdRatioBasedSampler(0.1)
                    ))
                    .AddOtlpExporter(otlp =>
                    {
                        otlp.Endpoint = new Uri(builder.Configuration["Telemetry:OtlpEndpoint"] ?? "http://otel-collector:4317");
                        otlp.Protocol = OtlpExportProtocol.Grpc;
                    });
            });
            
        return builder;
    }
}

/// <summary>
/// Custom sampler that ensures AI operations are always traced
/// </summary>
public class CustomAIEndpointSampler : Sampler
{
    /// <summary>
    /// Makes sampling decision based on request path
    /// </summary>
    /// <param name="samplingParameters">Parameters for sampling decision</param>
    /// <returns>Sampling result with decision</returns>
    public override SamplingResult ShouldSample(in SamplingParameters samplingParameters)
    {
        // Check baggage for HTTP path
        foreach (var attribute in samplingParameters.ParentContext.Baggage)
        {
            if (attribute.Key == "http.path")
            {
                string path = attribute.Value.ToString();
                
                // Always trace AI endpoints for detailed monitoring
                if (path?.StartsWith("/api/ai") == true ||
                    path?.Contains("/completions") == true ||
                    path?.Contains("/embeddings") == true)
                {
                    return new SamplingResult(SamplingDecision.RecordAndSample);
                }
                
                // Skip tracing for health checks to reduce noise
                if (path?.StartsWith("/health") == true)
                {
                    return new SamplingResult(SamplingDecision.Drop);
                }
            }
        }
        
        // Default to parent-based sampling
        return new SamplingResult(SamplingDecision.RecordAndSampled);
    }
}
```

## Conclusion and Next Steps

### Summary

This comprehensive documentation provides a solid foundation for building AI-powered applications using .NET Aspire and Semantic Kernel. By following the architecture patterns and implementation techniques outlined, you can create scalable, maintainable, and secure AI services that integrate seamlessly with your enterprise applications.

The key components we've covered include:

1. **Clean Architecture** with well-defined layers:
   - Domain models and interfaces
   - Data access with Entity Framework Core
   - Service layer with business logic
   - API endpoints with REST and gRPC support

2. **Semantic Kernel Integration** techniques:
   - Plugin development and registration
   - AI planning and orchestration
   - Prompt management and caching
   - Performance optimization

3. **Security and Enterprise Readiness**:
   - Authentication and authorization
   - API key management
   - Sensitive data handling
   - Content validation and sanitization

4. **Observability and Monitoring**:
   - Telemetry with OpenTelemetry
   - Health checks for reliability
   - Grafana dashboards for real-time monitoring
   - Structured logging with Serilog

### Next Steps in Your AI Journey

To continue evolving your AI-powered applications, consider these next steps:

1. **Advanced AI Planning Strategies**
   - Implement more complex task planning with FunctionCallingStepwisePlanner
   - Build domain-specific agents with specialized plugins
   - Create hybrid AI/rule-based systems for critical workflows

2. **Enhanced Natural Language Processing**
   - Add semantic search capabilities with vector databases
   - Implement Retrieval-Augmented Generation (RAG) patterns
   - Build conversational memory and context management

3. **Performance Optimization**
   - Implement model distillation for faster responses
   - Deploy with hardware acceleration (GPU support)
   - Use semantic caching for similar queries

4. **Continuous Improvement**
   - Add feedback loops for AI model improvement
   - Implement A/B testing for prompts and models
   - Build monitoring systems for content quality and accuracy

5. **Advanced Integration Scenarios**
   - Connect with enterprise data sources
   - Implement custom connectors for specialized systems
   - Build workflow automation with AI-driven decision making

### Getting Help and Contributing

This documentation is designed to evolve with the ecosystem. For updates, contributions, and community support:

1. **Official Resources**:
   - [Semantic Kernel GitHub](https://github.com/microsoft/semantic-kernel)
   - [.NET Aspire Documentation](https://learn.microsoft.com/en-us/dotnet/aspire/)
   - [Microsoft AI Platform](https://www.microsoft.com/ai)

2. **Community**:
   - [Semantic Kernel Discord Server](https://aka.ms/sk-discord)
   - [Stack Overflow - Semantic Kernel tag](https://stackoverflow.com/questions/tagged/semantic-kernel)
   - [.NET Community Forums](https://forums.dotnetfoundation.org/)

3. **Contributing**:
   - Report issues on the respective GitHub repositories
   - Submit pull requests for bug fixes and enhancements
   - Share your implementation patterns and success stories

By leveraging the patterns and practices outlined in this documentation, you're well on your way to building powerful, scalable, and maintainable AI applications with .NET Aspire and Semantic Kernel.

Happy coding!

---

**Authored by: Shawn Delaine Bellazan Jr**  
**Last Updated: 2025-03-17 20:13:22 UTC**

```