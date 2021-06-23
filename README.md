Simple ERC721 
ERC721 is a standard for representing ownership of non-fungible tokens where each token is unique such as in air tickets or real estates.
Here, we will create it and deploy it to a public ethereum testnet called Ropsten.

We will leverage on an open and well existing libraries called openZeppelin(set of well tested contracts) to create our ERC721 and deploy using the Remix IDE tool.

OpenZeppelin - A library for secure smart contract development. Build on a solid foundation of community-vetted code.
Remix IDE - A powerful open source tool that helps you write Solidity contracts straight from the browser.

Setting up the Environment
Open https://remix.ethereum.org/  in your browser preferably google chrome.
If you haven’t used Remix before, you need to setup plugins so that you can use the Solidity Compiler and Deploy and Run Transactions.

Select the Solidity button under the Environments heading and this will add the required plugins

Installation
Tatum Nuget version Tatum Nuget downloads
You can link this library as a standard Nuget package as described in the documentation.

Clients Usage
The only classes/interfaces used by the end user are those with suffix Client eg. IBitcoinClient, IEthereumClient, ITatumClient. They are located in Clients folder of the Tatum project. The only client not related to specific cryptocurrency/blockchain is the ITatumClient. ITatumClient enables to call Tatum API endpoints not related to the specific cryptocurency/blockchain.

Before There are 2 basic ways, how to use Clients.

Direct usage via Create Factory method
First you need to add Tatum.Clients namespace.

using Tatum.Clients;
Then just call Create() method

var bitcoinClient = BitcoinClient.Create(baseUrl, "your-x-api-key");
or

IBitcoinClient bitcoinClient = BitcoinClient.Create(baseUrl, "your-x-api-key");
where typically string baseUrl = "https://api-eu1.tatum.io";.
Of course you should store your credentials securely eg. in environment variable or configuration file.
Then you can call all the methods available for the Client. You can find examples of this approach in Tests project.

Usage via Dependency Injection
There is also IServiceCollection extension method AddTatum(...) for comfortable usage of .NET Core built in Dependency Injection (DI) container. You can read more about DI in .NET docs. If we follow the DI example from .NET docs, you can add all the Clients into DI like that.

using System.Threading.Tasks;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Tatum;

namespace DependencyInjection.Example
{
    class Program
    {
        static Task Main(string[] args) =>
            CreateHostBuilder(args).Build().RunAsync();

        static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureServices((_, services) =>
                    services.AddHostedService<Worker>()
                            .AddTatum("https://api-eu1.tatum.io", "your-x-api-key"));
    }
}
Then you can inject arbitrary client into your Worker service.

using System;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Extensions.Hosting;
using Tatum.Clients;

namespace DependencyInjection.Example
{
    public class Worker : BackgroundService
    {
        private readonly IBitcoinClient bitcoinClient;

        public Worker(IBitcoinClient bitcoinClient) =>
            this.bitcoinClient = bitcoinClient;

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            var info = await bitcoinClient.GetBlockchainInfo();
        }
    }
}
Wallet Static methods
As in case of Tatum JS library this library also provides static methods generating Wallet, Private Key and Address. You can find them in the Wallet class. They could be used as follows.

var wallet = Wallet.Create(Currency.BTC, mnemonic, testnet: true);
var address = Wallet.GenerateAddress(Currency.ETH, mnemonic, index: 1, testnet: true);
var privateKey = Wallet.GeneratePrivateKey(Currency.LTC, mnemonic, index: 1, testnet: true);
