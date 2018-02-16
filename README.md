# Ethereum workshop 2 : Adoção de animais para um Pet Shop 

#### NOTE: Swapy Network together with Ethereum-BH Meetup Group is hosting the second Ethereum-BH Workshop in Belo Horizonte, Brazil. This repository is based on [Pet Shop tutorial](http://truffleframework.com/tutorials/pet-shop) and aims to clarify all the process of creating a decentralized application. 

## Conteúdo

* [Introdução](#introducao)
* [Ambiente](#ambiente)
* [Desenvolvimento](#desenvolvimento)
  * [Estrutura de pastas](#estrutura-de-pastas)
  * [Escrevendo os Smart Contracts](#escrevendo-os-smart-contracts)
  
## Introdução

Nossa aplicação consiste em um sistema para o controle de adoções de animais em um Pet Shop e, para isso, esse tutorial aborda o processo de:

* Configurar o ambiente de desenvolvimento
* Escrever os smart contracts
* Compilar e publicar os contratos
* Testar os contratos 
* Criar uma interface web para o usuário
* Interagir com o aplicativo descentralizado através do browser

## Ambiente

Antes de começarmos a desenvolver nossa aplicação existem alguns requisistos técnicos. Instale o que segue:

* [Node.js v6+ LTS e npm](https://nodejs.org/en/) e npm (é instalado junto com o Node)
* [Git](https://git-scm.com/)

Após isso, nós precisamos somente instalar o Truffle, o framework que auxiliará no desenvolvimento do nosso DApp:

```
npm install -g truffle
```
Para verificar se o Truffle está instalado corretamente, digite ``` truffle version ``` no seu terminal. Caso veja algum erro, certifique-se que seus módulos npm estão visíveis nas variáveis de ambiente 

Utilizaremos também a Ganache, uma blockchain privada que permite a publicação dos nossos contratos em ambiente de desenvolvimento e dará suporte para o uso e teste da nossa aplicação. Faça o download da mesma em http://truffleframework.com/ganache.

## Desenvolvimento

### Estrutura de pastas

A estrutura inicial de pastas fornecida pelo Truffle possui:

* ```contracts/```: Contém o código fonte dos nossos contratos na linguagem [Solidity](https://solidity.readthedocs.io/en/develop/)
* ```migrations/```: Responsável por migrar nossos contratos para o ambiente desejado. O Truffle possui um sistema de controle de alterações.
* ```test/```: Contém os testes em Javascript e Solidity
* ```truffle.js```: Arquivo de configuração onde é descrito, por exemplo, o host dos contratos. Utilizaremos a rede local fornecida pela Ganache 

Esse repositório possui outras pastas que não importam no momento e serão descritas no decorrer do tutorial.

### Escrevendo os Smart Contracts

Nós começaremos nosso aplicativo descentralizado escrevendo os contratos, que atuam como o "back-end" e criam a interface para o armazenamento na blockchain.

Crie um novo arquivo de Adoption.sol na pasta contracts/ .

Adicione esse conteúdo ao arquivo:

```
pragma solidity ^0.4.17;

contract Adoption {

}
```
Observações:

* A versão mínima requerida do Solidity é descrita no ínicio do contrato: ```pragma solidity ^0.4.17;```. A palavra-chave ```pragma``` signfifica "informação adicional que importa somente ao compilador", enquanto o símbolo ```^``` significa "A versão indicada ou superior".
* Assim como em outras linguagens, a sintaxe exige ```;``` ao final de cada comando.


## Criando uma interface para interagir com o smart contract

Agora que nós criamos o smart contract, fizemos o deploy para nossa blockchain local e interagimos com ele via console, é hora de criar uma UI para que os donos de petshop possam utilizá-lo.

Junto com os smart contracts, nesse repositório, há uma pasta `src/` com o código para o front-end.

O font-end não usa nenhum sistema de build (webpack. grunt, etc) e nenhum framework ou biblioteca javascript (Angular, React, Vue, etc) para ser o mais simples possível para começar. A estrutura do app já está disponível nessa pasta; na próxima etapa do tutorial nós vamos preencher as funções relevantes para interagir com o Ethereum. Dessa forma, você pode pegar esse conhecimento e aplicar da forma como preferir no seu *workflow*.

### Instanciando o web3

1. Abra `src/js/app.js` no seu editor de texto.
2. Leia o arquivo. Note que há um objeto global `App` para gerenciar a aplicação, carregar os dados do petshot na função `init()` e chamar a função `initWeb3()`. A [biblioteca JavaScript web3](https://github.com/ethereum/web3.js/) interage com a blockchain do Ethereum. Ela pode acessar carteiras, assinar e enviar transações, interagir com os smart contracts, etc.
3. Remova o comentário de dentro de `initWeb3()` e cole o seguinte código:

```javascript
// Is there an injected web3 instance?
if (typeof web3 !== 'undefined') {
  App.web3Provider = web3.currentProvider;
} else {
  // If no injected web3 instance is detected, fall back to Ganache
  App.web3Provider = new Web3.providers.HttpProvider('http://localhost:7545');
}
web3 = new Web3(App.web3Provider);
```

Observações:
- Primeiro, nós checamos se há alguma instância do web3 ativa. Browsers Ethereum como o [Mist](https://github.com/ethereum/mist) ou a extensão [MetaMask](https://metamask.io/) injetam sua própria instância da web3. Se há alguma instância na `window`, nós pegamos o seu `provider` e usamos para criar a nossa instância da web3.
- Se nenhuma instância da web3 está presente, nós criamos a nossa usando o nosso provider local.

### Instanciando o contrato
Agora que nós podemos interagir com o Ethereum via web3, nós precisamos instanciar os nossos smart contracts para que o web3 saiba encontrá-lo e utilizá-lo. Nesse tutorial, estamos utilizando uma biblioteca do Truffle chamada [`truffle-contract`](https://github.com/trufflesuite/truffle-contract). Ela mantém as informações do contrato sincronizadas com as *migrations*, assim você não precisa alterar os endereços dos contratos manualmente.

Nessa etapa, é interessante entender como o web3 consegue entender o smart contract e  se comunicar com ele: ao compilar o código solidity um arquivo `.json` é criado. Esse arquivo é chamado de [ABI (Application Binary Interface)](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI). Ele contém metadados do contrato, como as suas variáveis, sua funções e os respectivos tipos e retornos. Dessa forma, se o seu contrato possui a seguinte função: 

```solidity
function sum(uint x, uint y) public returns (uint) {
  return x + y;
}
```

Através da ABI, o web3 vai entender que o contrato possui uma função `sum` que deve receber dois inteiros por parâmetro e retornar outro inteiro:

```javascript
myContractInstance.sum.call(x, y, (error, result) => {
  assert(!error);
  assert(result).equals(x + y);
});
```

Prosseguindo com o tutorial:
1. Ainda no arquivo `src/js/app.js`, remova o bloco de comentários de dentro da função `initContract()` e inclua o seguinte trecho de código:

```javascript
$.getJSON('Adoption.json', function(data) {
  // Get the necessary contract artifact file and instantiate it with truffle-contract
  var AdoptionArtifact = data;
  App.contracts.Adoption = TruffleContract(AdoptionArtifact);

  // Set the provider for our contract
  App.contracts.Adoption.setProvider(App.web3Provider);

  // Use our contract to retrieve and mark the adopted pets
  return App.markAdopted();
});
```

Observações:

- Primeiro pegamos a ABI do contrato.
- Após recebermos o JSON no callback, nós o passamos para `TruffleContract()`. Isso irá criar a instância do contrato com a qual podemos interagir.
- Com o nosso contrato instanciado, nós setamos o seu *web3 provider* usando o `App.web3Provider` que havíamos iniciado quando configuramos o web3.
- Depois nós chamamos a função `markAdopted()` para marcar os pets que ja foram adotados anteriormente. Nós encapsulamos isso em uma função separada já que vamos precisar de atualizar a UI toda vez que fizermos alguma modificação na blockchain.

### Consultando os pets adotados e atualizando a UI

1. Ainda em `src/js/app.js`, remova o bloco de comentário em `markAdopted` e altere para:

```javascript
var adoptionInstance;

App.contracts.Adoption.deployed().then(function(instance) {
  adoptionInstance = instance;

  return adoptionInstance.getAdopters.call();
}).then(function(adopters) {
  for (i = 0; i < adopters.length; i++) {
    if (adopters[i] !== '0x0000000000000000000000000000000000000000') {
      $('.panel-pet').eq(i).find('button').text('Success').attr('disabled', true);
    }
  }
}).catch(function(err) {
  console.log(err.message);
});
```

Observações:
- Nós acessamos o contrato `Adoption` que foi feito deploy em nossa blockchain local e chamamos `getAdopters` nessa instância.
- Primeiro, nós declaramos a variável `adoptionInstance` fora da chamada do smart contract, para que possamos acessar a instância depois de recuperá-la da blockchain.
- Usar a função `call()` nos permite ler os dados da blockchain sem alterar o seu estado, ou seja, sem enviar uma "transação completa". Dessa forma, não gastamos gas.
- Após chamar `getAdopters()`, nós fazemos um loop para checar se há algum endereço correspondente a posição.
- Quando há ....

### Função adopt()

....

## Interagindo com o dapp no browser

Agora estmoas prontos para usar o dapp!

### Instalando e configurando o MetaMask

O jeito mais fácil de interagir com o nosso dapp no browser é utilizando a extensão [MetaMask](https://metamask.io).

1. Instale o MetaMask no seu browser
2. Após finalizar a instalação, você verá o ícone da raposa próximo a barra de endereço. Clique nele e você verá essa tela:

![Privacy note](http://truffleframework.com/tutorials/images/pet-shop/metamask-privacy.png)

3. Clique em `Accept` para aceitar a *Privacy Notice*.
4. Após isso você verá os Termos de Uso. Leia-os, role até o final e clique em `Accept`.

![Terms of use](http://truffleframework.com/tutorials/images/pet-shop/metamask-terms.png)

5. Agora você verá a tela inicial o MetaMask. Clique em **Import Existing DEN**.

![Tela inicial do MetaMask](http://truffleframework.com/tutorials/images/pet-shop/metamask-initial.png)

6. Na caixa de texto `Wallet Seed`, coloque o *mnemônico* que é mostrado no ganache

> candy maple cake sugar pudding cream honey rich smooth crumble sweet treat

```
WARNING:
Não use esse mnemônico na mainnet. Se você enviar ETH para qualquer conta criada com esse mnemônico você provavelmente irá perder tudo!
```
Digite uma senha abaixo e clique em **OK**.

![Seed](http://truffleframework.com/tutorials/images/pet-shop/metamask-seed.png)

7. Agora precisamos conectar o MetaMask na blockchain do Ganache. Clique no menu que mostra `Main Network` e selecione **Custom RPC**.


![Menu de networks do MetaMask](http://truffleframework.com/tutorials/images/pet-shop/metamask-networkmenu.png)

8. Na caixa de texto chamada "New RPC URL" digite `http://127.0.0.1:7545` e clique em **Save**.

![MetaMask Custom RPC](http://truffleframework.com/tutorials/images/pet-shop/metamask-customrpc.png)


O nome da network no topo irá mudar para "Private Network".

9. Clique na seta para retornar para a página de "Accounts".

Cada carteira criada com o comando Truffle Develop possui 100 ether. Você irá perceber um pouco menos do que isso na primeira conta, porque para fazer o deploy do contrato e rodar os testes, um pouco de ether foi gasto.

![Conta do MetaMask](http://truffleframework.com/tutorials/images/pet-shop/metamask-account1.png)

A configuração está completa agora!

## Iniciando o servidor http local

O repositório está configurado com o pacote `lite-server` que irá iniciar um servidor HTTP estático em `localhost:3000`.

Para iniciá-lo utilize o comando `npm run dev`.

## Usando o dapp
1. Rode o servidor local com `npm run dev`.

O servior irá iniciar e abrir automaticamente uma nova tab no seu browser com o dapp.

![Pet Shop](http://truffleframework.com/tutorials/images/pet-shop/dapp.png)

Automaticamente, o MetaMask irá abrir um popup contendo informações sobre a transação a ser feita. Clique em **Submit** para aprovar a transação.

![Informações da transação](http://truffleframework.com/tutorials/images/pet-shop/metamask-transactionconfirm.png)

Após isso, você verá a transação listada na sua conta do MetaMask

![Transações do MetaMask](http://truffleframework.com/tutorials/images/pet-shop/metamask-transactionsuccess.png). 

