# Validador de CPF - Function App na Azure

Este projeto implementa uma aplicação para validação de CPF utilizando o Azure Function App. Foi desenvolvido como parte do bootcamp **Microsoft Certification Challenge #2 AZ-240** pela Digital Innovation One (DIO).

## Visão Geral

O projeto consiste em uma função HTTP Trigger que recebe um CPF como entrada e realiza a validação do mesmo, retornando se é válido ou inválido. A validação segue as regras oficiais para CPFs no Brasil.

## Tecnologias Utilizadas

- **C#**: Linguagem utilizada no desenvolvimento.
- **Azure Functions**: Plataforma de computação serverless para execução da função.
- **Microsoft.AspNetCore.Mvc**: Framework para construir aplicações web e APIs.
- **Newtonsoft.Json**: Biblioteca para manipulação de JSON.

## Estrutura do Projeto

O código principal está implementado na função `fnvalidacpf`, localizada no arquivo `fnvalidacpf.cs`. A função recebe requisições HTTP do tipo POST, valida o CPF e retorna o resultado.

### Código Principal

```csharp
[FunctionName("fnvalidacpf")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
    ILogger log)
{
    log.LogInformation("Iniciando a validação do CPF.");

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    if (data == null)
    {
        return new BadRequestObjectResult("Por favor, informe o CPF.");
    }

    string cpf = data?.cpf;
    if (string.IsNullOrEmpty(cpf) || !ValidaCPF(cpf))
    {
        return new BadRequestObjectResult("CPF inválido.");
    }

    return new OkObjectResult("CPF válido.");
}

public static bool ValidaCPF(string cpf)
{
    if (cpf.Length != 11 || !Regex.IsMatch(cpf, @"^\d{11}$"))
        return false;

    if (new string(cpf[0], cpf.Length) == cpf)
        return false;

    int[] multiplicador1 = new int[9] { 10, 9, 8, 7, 6, 5, 4, 3, 2 };
    int[] multiplicador2 = new int[10] { 11, 10, 9, 8, 7, 6, 5, 4, 3, 2 };

    string tempCpf = cpf.Substring(0, 9);
    int soma = 0;

    for (int i = 0; i < 9; i++)
        soma += int.Parse(tempCpf[i].ToString()) * multiplicador1[i];

    int resto = soma % 11;
    if (resto < 2)
        resto = 0;
    else
        resto = 11 - resto;

    string digito = resto.ToString();
    tempCpf = tempCpf + digito;
    soma = 0;

    for (int i = 0; i < 10; i++)
        soma += int.Parse(tempCpf[i].ToString()) * multiplicador2[i];

    resto = soma % 11;
    if (resto < 2)
        resto = 0;
    else
        resto = 11 - resto;

    digito = digito + resto.ToString();

    return cpf.EndsWith(digito);
}
```

## Como Utilizar

### Pré-requisitos

1. Uma conta no Azure.
2. Instalação do [Azure Functions Core Tools](https://learn.microsoft.com/en-us/azure/azure-functions/functions-run-local).
3. Ambiente de desenvolvimento configurado com .NET.

### Passos para Execução

1. **Clone o repositório:**
   ```bash
   git clone <URL_DO_REPOSITORIO>
   ```

2. **Publique a função no Azure:**
   Utilize o Azure Functions Core Tools ou o Visual Studio para fazer o deploy.

3. **Faça uma requisição:**
   - Utilize uma ferramenta como o Postman ou Curl para enviar uma requisição POST.
   - Endpoint: `https://<NOME_DA_SUA_FUNCTION_APP>.azurewebsites.net/api/fnvalidacpf`
   - Body (JSON):
     ```json
     {
       "cpf": "12345678909"
     }
     ```

4. **Receba a resposta:**
   - Para CPF válido: `"CPF válido."`
   - Para CPF inválido: `"CPF inválido."`

## Contribuições

Contribuições são bem-vindas! Sinta-se à vontade para abrir issues ou pull requests para melhorias.

## Licença

Este projeto é licenciado sob a [MIT License](LICENSE).

---

**Autor:** Desenvolvido como parte do bootcamp DIO - Microsoft Certification Challenge #2 AZ-240.
Luiz Fabiano de Souza ~ LsouzaDev Back-End Developer

