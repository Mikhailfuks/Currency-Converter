using System;
using System.Net.Http;
using System.Net.Http.Json;
using System.Threading.Tasks;
using System.Text.Json;

public class CurrencyConverter
{
    private static readonly HttpClient httpClient = new HttpClient();
    private static readonly string apiUrl = "https://api.exchangeratesapi.io/latest"; //Free API - usage limits apply!


    public static async Task Main(string[] args)
    {
        Console.Write("Enter the amount to convert: ");
        string amountStr = Console.ReadLine();

        Console.Write("Enter the source currency (e.g., USD): ");
        string sourceCurrency = Console.ReadLine().ToUpper();

        Console.Write("Enter the target currency (e.g., EUR): ");
        string targetCurrency = Console.ReadLine().ToUpper();

        if (!double.TryParse(amountStr, out double amount))
        {
            Console.WriteLine("Invalid amount.");
            return;
        }

        try
        {
            double convertedAmount = await ConvertCurrency(amount, sourceCurrency, targetCurrency);
            Console.WriteLine($"{amount} {sourceCurrency} is equal to {convertedAmount:F2} {targetCurrency}");
        }
        catch (HttpRequestException ex)
        {
            Console.WriteLine($"Error fetching exchange rates: {ex.Message}");
        }
        catch (JsonException ex)
        {
          Console.WriteLine($"Error parsing JSON response: {ex.Message}");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"An error occurred: {ex.Message}");
        }
    }


    public static async Task<double> ConvertCurrency(double amount, string sourceCurrency, string targetCurrency)
    {
        HttpResponseMessage response = await httpClient.GetAsync($"{apiUrl}?base={sourceCurrency}");
        response.EnsureSuccessStatusCode(); // Throw an exception for bad status codes

        var exchangeRates = await response.Content.ReadFromJsonAsync<ExchangeRates>();

        if (exchangeRates == null || !exchangeRates.rates.ContainsKey(targetCurrency))
        {
            throw new Exception($"Exchange rate for {targetCurrency} not found.");
        }

        return amount * exchangeRates.rates[targetCurrency];
    }
}


//Helper class to deserialize JSON response
public class ExchangeRates
{
    public string baseCurrency { get; set; }
    public DateTime date { get; set; }
    public Dictionary<string, double> rates { get; set; } = new Dictionary<string, double>();

}

