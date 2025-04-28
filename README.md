# Android Crypto Monitor 📈

Aplicativo Android para monitoramento de preços de criptomoedas em tempo real.

---

## Demonstração 🎥

Veja abaixo uma demonstração do funcionamento do aplicativo:

![crypto-monitor-animaçao](https://github.com/user-attachments/assets/5295f95e-7d1b-4bad-8880-88ebcea40b4e)



---

## 📱 Sobre o Projeto

O Android Crypto Monitor é um aplicativo simples que busca e exibe informações atualizadas de diferentes criptomoedas.  
Ele conecta-se a uma API pública de criptomoedas, obtém os preços mais recentes e exibe esses dados em uma lista clara e intuitiva.

Ideal para quem quer acompanhar o mercado de criptoativos de forma prática, diretamente pelo celular.

## Como Funciona o Aplicativo ⚙️

O Android Crypto Monitor conecta-se a APIs de mercado de criptomoedas para buscar dados atualizados sobre diferentes moedas. O aplicativo exibe essas informações em tempo real para o usuário, permitindo acompanhar flutuações de preço de maneira rápida e prática.

Fluxo básico:
- O app inicia e realiza uma chamada de rede para buscar as informações de moedas.
- Os dados são processados e exibidos na tela principal.
- O usuário visualiza informações como nome da moeda e valor atualizado.

---

## Estrutura de Código 🛠️

### 1. Service (MercadoBitcoinService.kt)

Essa parte define uma interface em Kotlin que descreve como o aplicativo irá se comunicar com a API do Mercado Bitcoin.


1.1 interface *MercadoBitcoinService*
```kotlin
interface MercadoBitcoinService {
````
- interface em Kotlin é como um contrato:
Ela define o que deve ser feito, mas não implementa o como.

- Neste caso, é uma interface para o Retrofit:
Ela diz qual requisição HTTP o app vai fazer.


1.2 *@GET("api/BTC/ticker/")*
```kotlin
    @GET("api/BTC/ticker/")
````
- Essa é uma anotação do Retrofit.

- Ela informa que:

  - Vamos fazer uma requisição HTTP GET.

  - O endereço será o final da URL: *api/BTC/ticker/*.

  - Base URL (definida no *MercadoBitcoinServiceFactory*) + este caminho → vira a URL completa da requisição.

1.3 *suspend fun getTicker(): Response<TickerResponse>*
```kotlin
    suspend fun getTicker(): Response<TickerResponse>
}
````

- Define como o app conversa com a API:
- Ele faz uma requisição GET para buscar o preço atual do Bitcoin de forma rápida e sem travar a tela!

### 2. Factory (ServiceFactory.kt)

Ela é responsável por criar e configurar o cliente de rede que vai se conectar à API do Mercado Bitcoin.

2.1 *class MercadoBitcoinServiceFactory*

````kotlin
class MercadoBitcoinServiceFactory {
````
- Define uma classe normal chamada *MercadoBitcoinServiceFactory*.

- Ela serve para construir um objeto que poderá fazer requisições HTTP para buscar informações da API do Mercado Bitcoin.


2.2 Método *create()*
   
````kotlin

    fun create(): MercadoBitcoinService {
        val retrofit = Retrofit.Builder()
            .baseUrl("https://www.mercadobitcoin.net/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()

        return retrofit.create(MercadoBitcoinService::class.java)
    }
}
````
- Esse método:

  - Cria uma instância do Retrofit (biblioteca que facilita chamadas HTTP em Android).

- Configura o Retrofit para se comunicar com o site https://www.mercadobitcoin.net/.

- Define o GsonConverterFactory, que serve para transformar automaticamente a resposta JSON da API em objetos Kotlin.

- No final, retorna um serviço que segue o contrato definido pela interface *MercadoBitcoinService*.


2.3 Dentro do *create()*

````kotlin

val retrofit = Retrofit.Builder()
    .baseUrl("https://www.mercadobitcoin.net/")
    .addConverterFactory(GsonConverterFactory.create())
    .build()

````

- *Retrofit.Builder()* → Começa a construir um novo cliente de rede.

- *.baseUrl(...)* → Define a URL base onde o app vai fazer as chamadas (a raiz da API).

- *.addConverterFactory(GsonConverterFactory.create())* → Configura o Retrofit para converter o JSON da resposta automaticamente para objetos Kotlin usando Gson.

- *.build()* → Finaliza a construção e cria a instância do Retrofit.

Depois: 


````kotlin

return retrofit.create(MercadoBitcoinService::class.java)

````

- Cria um objeto que implementa a interface *MercadoBitcoinService*, que descreve quais endpoints da API o app pode chamar.

- Esse serviço é usado, por exemplo, no método *makeRestCall()* da *MainActivity* para buscar o preço da criptomoeda.


### 3. Main (MainActivity.kt)

Esta classe é a tela principal do aplicativo Android.

Ela é responsável por:

- Configurar a interface gráfica (Toolbar e botão Refresh).

- Fazer chamadas REST para buscar o preço de uma criptomoeda (no caso, do Mercado Bitcoin).

- Atualizar as informações na tela.

- Tratar possíveis erros.

3.1 Método onCreate

````kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
````
Esse é o ponto inicial da tela.
O Android chama esse método automaticamente quando a *MainActivity* é criada.

Dentro do onCreate, são feitas três coisas:

- *setContentView(R.layout.activity_main)*
→ Define qual layout XML será usado para desenhar a tela (*activity_main.xml*).

- Configuração da Toolbar

````kotlin
        // Configurando a toolbar
        val toolbarMain: Toolbar = findViewById(R.id.toolbar_main)
        configureToolbar(toolbarMain)
````
→ Localiza a toolbar no layout e chama um método para configurar ela (cores, título etc).

- Configuração do Botão Refresh

````kotlin
        // Configurando o botão Refresh
        val btnRefresh: Button = findViewById(R.id.btn_refresh)
        btnRefresh.setOnClickListener {
            makeRestCall()
        }
    }
````
→ Localiza o botão de atualizar no layout e define que, quando clicado, o app fará uma nova chamada REST para buscar dados atualizados.

3.2 Método *configureToolbar*

````kotlin 
    private fun configureToolbar(toolbar: Toolbar) {
        setSupportActionBar(toolbar)
        toolbar.setTitleTextColor(getColor(R.color.white))
        supportActionBar?.setTitle(getText(R.string.app_title))
        supportActionBar?.setBackgroundDrawable(getDrawable(R.color.primary))
    }
````
Esse método:

- Define a Toolbar como ActionBar da Activity.

- Altera a cor do texto do título para branco.

- Define o título do app.

- Define a cor de fundo da Toolbar.

Ou seja, ele estiliza a parte de cima do app para ficar personalizada.

3.3 Método *makeRestCall*

````kotlin 
    private fun makeRestCall() {
        CoroutineScope(Dispatchers.Main).launch {
            try {
                val service = MercadoBitcoinServiceFactory().create()
                val response = service.getTicker()

                if (response.isSuccessful) {
                    val tickerResponse = response.body()

                    // Atualizando os componentes TextView
                    val lblValue: TextView = findViewById(R.id.lbl_value)
                    val lblDate: TextView = findViewById(R.id.lbl_date)

                    val lastValue = tickerResponse?.ticker?.last?.toDoubleOrNull()
                    if (lastValue != null) {
                        val numberFormat = NumberFormat.getCurrencyInstance(Locale("pt", "BR"))
                        lblValue.text = numberFormat.format(lastValue)
                    }

                    val date = tickerResponse?.ticker?.date?.let { Date(it * 1000L) }
                    val sdf = SimpleDateFormat("dd/MM/yyyy HH:mm:ss", Locale.getDefault())
                    lblDate.text = sdf.format(date)

                } else {
                    // Trate o erro de resposta não bem-sucedida
                    val errorMessage = when (response.code()) {
                        400 -> "Bad Request"
                        401 -> "Unauthorized"
                        403 -> "Forbidden"
                        404 -> "Not Found"
                        else -> "Unknown error"
                    }
                    Toast.makeText(this@MainActivity, errorMessage, Toast.LENGTH_LONG).show()
                }

            } catch (e: Exception) {
                // Trate o erro de falha na chamada
                Toast.makeText(this@MainActivity, "Falha na chamada: ${e.message}", Toast.LENGTH_LONG).show()
            }
        }
    }
}
````
Aqui é onde acontece a parte mais importante: buscar os dados da API.

O que ele faz:

- Cria uma Coroutine no *Dispatchers.Main* (porque vai atualizar a tela depois).

- Chama o serviço da API usando o *MercadoBitcoinServiceFactory()*.

- Recebe a resposta:

  - Se a resposta for bem-sucedida (*response.isSuccessful*):

    - Pega o valor da última cotação (*last*) e atualiza o TextView *lblValue* com o valor formatado em R$ (Real).

    - Pega a data do preço (*date*) e formata ela para o padrão brasileiro (*dd/MM/yyyy HH:mm:ss*), exibindo no TextView *lblDate*.

  - Se a resposta não for bem-sucedida:

    - Exibe uma mensagem de erro correspondente (*404 = Not Found, 401 = Unauthorized*, etc) usando um *Toast*.

  - Se ocorrer alguma exceção (como erro de rede):

    - Exibe um Toast avisando que houve uma falha e mostra a mensagem de erro.




### 4. Model (TicketResponse.kt)

Eles representam o formato da resposta que o app recebe da API quando faz a requisição para buscar o preço da criptomoeda.

````kotlin
class TickerResponse(
    val ticker: Ticker
)
````
- Essa classe representa a resposta inteira que vem da API.

- Dentro dessa resposta, existe um objeto chamado ticker.

- Esse objeto ticker é da classe Ticker, que contém os detalhes da criptomoeda (preços, volume, data etc).

````kotlin
class Ticker(
    val high: String,
    val low: String,
    val vol: String,
    val last: String,
    val buy: String,
    val sell: String,
    val date: Long
)
````
- Essa classe representa todos os dados de um único Ticker (uma moeda).

- Ela tem vários atributos:


Significado de cada atributo

*high*:	Maior preço da moeda no período (ex: 70.000)
*low*:	Menor preço da moeda no período (ex: 65.000)
*vol*:	Volume de moedas negociadas
*last*:	Último preço de negociação (preço atual)
*buy*:	Preço de compra
*sell*:	Preço de venda
*date*:	Data/hora da cotação em timestamp (segundos desde 1970)

- Exemplo prático:

  - Se você pedir os dados do Bitcoin agora, o campo *last* vai ter o preço atual do Bitcoin em reais.

    - *high* e *low* mostram o quanto o preço variou no dia.

    - *vol* mostra quanto de Bitcoin foi negociado.

---


## Como Rodar o Projeto 🧑‍💻

1. Clone o repositório:
   ```bash
   git clone https://github.com/carreiras/android-crypto-monitor.git

2. Abra o projeto no Android Studio.

3. Sincronize o Gradle.

4. Execute o projeto em um emulador Android ou dispositivo real.

   

## Tecnologias Utilizadas 🚀

- [Kotlin](https://kotlinlang.org/) — Linguagem principal utilizada no desenvolvimento do app.
- [Retrofit](https://square.github.io/retrofit/) — Biblioteca para consumo de APIs HTTP.
- [Android SDK](https://developer.android.com/studio) — Kit de desenvolvimento para aplicações Android.
- [Gradle](https://gradle.org/) — Sistema de automação de builds.
- [Android Studio](https://developer.android.com/studio) — IDE oficial para desenvolvimento Android ([Clique aqui para baixar](https://developer.android.com/studio)).
