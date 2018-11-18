using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using PM.Server;
using PM.Server.Extensions;
using Swashbuckle.AspNetCore.Swagger;
using System.Threading.Tasks;
using System.Net;
using PM.Server.SignalR;
using OpenIddict.Core;
using System;
using System.Threading;
using OpenIddict.Models;
using Microsoft.AspNetCore.HttpOverrides;
using Microsoft.AspNetCore.Rewrite;
using PMAPI.Server.Extensions;
using Microsoft.Extensions.FileProviders;
using Microsoft.AspNetCore.Http;
using System.IO;
using Microsoft.AspNetCore.ResponseCompression;
using System.IO.Compression;

namespace PM
{
    public class Startup
    {
        // Order or run
        //1) Constructor
        //2) Configure services
        //3) Configure

        public static IHostingEnvironment _hostingEnv;
        public Startup(IConfiguration configuration, IHostingEnvironment env)
        {
            Configuration = configuration;
            _hostingEnv = env;

            Helpers.SetupSerilog();

            // var builder = new ConfigurationBuilder()
            //                .SetBasePath(env.ContentRootPath)
            //                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
            //                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
            //                .AddEnvironmentVariables();
            // if (env.IsDevelopment())
            // {
            //     // For more details on using the user secret store see http://go.microsoft.com/fwlink/?LinkID=532709
            //     builder.AddUserSecrets<Startup>();
            // }

            // Configuration = builder.Build();
        }

        public static IConfiguration Configuration { get; set; }
        // This method gets called by the runtime. Use this method to add services to the container.
        // For more information on how to configure your application, visit http://go.microsoft.com/fwlink/?LinkID=398940
        public void ConfigureServices(IServiceCollection services)
        {
            //services.AddCors(option =>
            //{
            //    option.AddPolicy("all", policy =>
            //    {
            //        policy.AllowAnyOrigin();
            //        policy.AllowAnyHeader();
            //        policy.WithOrigins("https://suggest.dxpapi.com", "https://suggest.dxpapi.com/api");
            //    });
            //});

            services.AddCustomHeaders();

            if (_hostingEnv.IsDevelopment())
            {
                services.AddSslCertificate(_hostingEnv, Configuration);
            }
            services.AddOptions();

            services.AddResponseCompression(options =>
            {
                options.Providers.Add<GzipCompressionProvider>();
                options.MimeTypes = Helpers.DefaultMimeTypes;
            });

            services.Configure<GzipCompressionProviderOptions>(options =>
            {
                options.Level = CompressionLevel.Fastest;
            });

            services.AddCustomDbContext();

            services.AddCustomIdentity();

            //          services.AddCustomOpenIddict();

            services.AddMemoryCache();

            services.RegisterCustomServices();

            //         services.AddSignalR();

            services.AddCustomLocalization();

            services.AddCustomizedMvc(Configuration);

            // Node services are to execute any arbitrary nodejs code from .net
            services.AddNodeServices();

            services.AddSwaggerGen(c =>
            {
                c.SwaggerDoc("v1", new Info { Title = "Papermart", Version = "v1" });
            });
//            services.AddRouting(options => options.LowercaseUrls = true);
        }
        public void Configure(IApplicationBuilder app)
        {
            app.UseCustomisedCsp();

            app.UseCustomisedHeadersMiddleware();

            app.AddCustomLocalization();

            app.AddDevMiddlewares();

            app.UseResponseCompression();

            //if (_hostingEnv.IsProduction())
            //{
            //    app.UseResponseCompression();
            //}

            //        app.SetupMigrations();

            // https://github.com/openiddict/openiddict-core/issues/518
            // And
            // https://github.com/aspnet/Docs/issues/2384#issuecomment-297980490

            var forwarOptions = new ForwardedHeadersOptions
            {
                ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto
            };
            forwarOptions.KnownNetworks.Clear();
            forwarOptions.KnownProxies.Clear();

            app.UseForwardedHeaders(forwarOptions);

            //          app.UseAuthentication();

            app.UseRewriter(new RewriteOptions()
                .Add(new RedirectLowerCaseRule())
                .AddRedirectToHttps()
               );

            //app.UseDefaultFiles();
            app.UseStaticFiles();
            /*
             * if we want to add max-age instead of version
                        app.UseStaticFiles(new StaticFileOptions
                        {
                            OnPrepareResponse = ctx =>
                            {
                                // Requires the following import:
                                // using Microsoft.AspNetCore.Http;
                                ctx.Context.Response.Headers.Append("Cache-Control", "public,max-age=600");
                            }
                        });
            */


            //app.UseFileServer(new FileServerOptions
            //{
            //    FileProvider = new PhysicalFileProvider(Configuration["AppSettings:ItemImages"].ToString()),
            //    RequestPath = new PathString("/CustomItem"),
            //    EnableDirectoryBrowsing = false
            //});

            //setting up item images folder
            //app.UseFileServer(new FileServerOptions
            //{
            //    FileProvider = new PhysicalFileProvider(Configuration["AppSettings:ItemImages"].ToString()),
            //    EnableDirectoryBrowsing = false
            //});

            app.UseStaticFiles(new StaticFileOptions
            {
                FileProvider = new PhysicalFileProvider(Configuration["AppSettings:ItemImages"]),
            });

            app.UseMvc(routes =>
            {
                // http://stackoverflow.com/questions/25982095/using-googleoauth2authenticationoptions-got-a-redirect-uri-mismatch-error
                //routes.MapRoute(name: "signin-google", template: "signin-google", defaults: new { controller = "Account", action = "ExternalLoginCallback" });

                //         routes.MapRoute(name: "externallogin", template: "externallogin", 
                //           defaults: new { controller = "externallogin", action = "Index" });

                routes.MapRoute(name: "pagenotfound", template: "pagenotfound", defaults: new { controller = "home", action = "PageNotFound" });

                routes.MapRoute(name: "set-language", template: "setlanguage", defaults: new { controller = "Home", action = "SetLanguage" });

                routes.MapSpaFallbackRoute(name: "spa-fallback", defaults: new { controller = "Home", action = "Index" });
            });

            //app.UseSpa(spa =>
            //{
            //    spa.Options.SourcePath = "ClientApp";

            //    /*
            //    // If you want to enable server-side rendering (SSR),
            //    // [1] In AspNetCoreSpa.csproj, change the <BuildServerSideRenderer> property
            //    //     value to 'true', so that the SSR bundle is built during publish
            //    // [2] Uncomment this code block
            //    */

            //    //   spa.UseSpaPrerendering(options =>
            //    //    {
            //    //        options.BootModulePath = $"{spa.Options.SourcePath}/dist-server/main.bundle.js";
            //    //        options.BootModuleBuilder = env.IsDevelopment() ? new AngularCliBuilder(npmScript: "build:ssr") : null;
            //    //        options.ExcludeUrls = new[] { "/sockjs-node" };
            //    //        options.SupplyData = (requestContext, obj) =>
            //    //        {
            //    //          //  var result = appService.GetApplicationData(requestContext).GetAwaiter().GetResult();
            //    //          obj.Add("Cookies", requestContext.Request.Cookies);
            //    //        };
            //    //    });

            //    if (_hostingEnv.IsDevelopment())
            //    {
            //           spa.UseAngularCliServer(;npmScript: "start");
            //        //   OR
            //        //spa.UseProxyToSpaDevelopmentServer("http://localhost:4200");
            //    }
            //});

        }
    }
}
