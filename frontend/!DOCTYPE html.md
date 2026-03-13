<!DOCTYPE html>  
<html>  
<head>  
    <title>Calculadora de Frete China → Brasil</title>  
    <meta charset="UTF-8">  
    <meta name="viewport" content="width=device-width, initial-scale=1.0">  
    <style>  
        body {  
            font-family: Arial, sans-serif;  
            max-width: 500px;  
            margin: 0 auto;  
            padding: 20px;  
            background: #f5f5f5;  
        }  
        .container {  
            background: white;  
            padding: 20px;  
            border-radius: 10px;  
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);  
        }  
        h1 {  
            color: #333;  
            text-align: center;  
        }  
        input, select, button {  
            width: 100%;  
            padding: 12px;  
            margin: 8px 0;  
            border: 1px solid #ddd;  
            border-radius: 5px;  
            box-sizing: border-box;  
        }  
        button {  
            background: #007bff;  
            color: white;  
            border: none;  
            font-size: 16px;  
            cursor: pointer;  
        }  
        button:hover {  
            background: #0056b3;  
        }  
        button:disabled {  
            background: #ccc;  
            cursor: not-allowed;  
        }  
        .resultado {  
            margin-top: 20px;  
            padding: 15px;  
            background: #e8f4fd;  
            border-radius: 5px;  
            display: none;  
        }  
        .rota {  
            border-bottom: 1px solid #ddd;  
            padding: 10px 0;  
        }  
        .rota:last-child {  
            border-bottom: none;  
        }  
        .erro {  
            color: red;  
            text-align: center;  
        }  
        .carregando {  
            text-align: center;  
            display: none;  
        }  
    </style>  
</head>  
<body>  
    <div class="container">  
        <h1>📦 Calculadora de Frete</h1>  
        <h3>China → Brasil</h3>  
          
        <div id="formulario">  
            <select id="categoria">  
                <option value="">Carregando categorias...</option>  
            </select>  
              
            <input type="number" id="peso" placeholder="Peso (kg)" step="0.1" min="0.1">  
            <input type="number" id="comprimento" placeholder="Comprimento (cm)" min="1">  
            <input type="number" id="largura" placeholder="Largura (cm)" min="1">  
            <input type="number" id="altura" placeholder="Altura (cm)" min="1">  
              
            <button onclick="calcular()" id="botao">Calcular Frete</button>  
        </div>  
          
        <div class="carregando" id="carregando">  
            ⏳ Calculando...  
        </div>  
          
        <div class="resultado" id="resultado">  
            <h3>📋 Opções de frete:</h3>  
            <div id="rotas"></div>  
        </div>  
          
        <div class="erro" id="erro"></div>  
    </div>  
  
    <script>  
        // Quando a página carregar, busca as categorias  
        window.onload = async function() {  
            try {  
                const resposta = await fetch('/api/produtos')  
                const dados = await resposta.json()  
                  
                const select = document.getElementById('categoria')  
                select.innerHTML = '<option value="">Selecione a categoria</option>'  
                  
                dados.tipos.forEach(tipo => {  
                    select.innerHTML += `<option value="${tipo.id}">${tipo.nome}</option>`  
                })  
            } catch (erro) {  
                document.getElementById('categoria').innerHTML = '<option value="">Erro ao carregar</option>'  
            }  
        }  
  
        // Função principal de calcular  
        async function calcular() {  
            // Pega valores  
            const categoria = document.getElementById('categoria').value  
            const peso = document.getElementById('peso').value  
            const comprimento = document.getElementById('comprimento').value  
            const largura = document.getElementById('largura').value  
            const altura = document.getElementById('altura').value  
              
            // Valida se tudo foi preenchido  
            if (!categoria || !peso || !comprimento || !largura || !altura) {  
                alert('Preencha todos os campos!')  
                return  
            }  
              
            // Mostra "carregando" e esconde resultados antigos  
            document.getElementById('carregando').style.display = 'block'  
            document.getElementById('resultado').style.display = 'none'  
            document.getElementById('erro').innerHTML = ''  
            document.getElementById('botao').disabled = true  
              
            try {  
                // Chama nossa API  
                const resposta = await fetch('/api/frete', {  
                    method: 'POST',  
                    headers: { 'Content-Type': 'application/json' },  
                    body: JSON.stringify({  
                        peso: peso,  
                        comprimento: comprimento,  
                        largura: largura,  
                        altura: altura,  
                        tipoProduto: categoria  
                    })  
                })  
                  
                const dados = await resposta.json()  
                  
                // Esconde "carregando"  
                document.getElementById('carregando').style.display = 'none'  
                  
                // Se deu erro  
                if (!dados.sucesso) {  
                    document.getElementById('erro').innerHTML = 'Erro: ' + dados.erro  
                    return  
                }  
                  
                // Se não tem rotas  
                if (!dados.rotas || dados.rotas.length === 0) {  
                    document.getElementById('erro').innerHTML = 'Nenhuma rota encontrada'  
                    return  
                }  
                  
                // Mostra as rotas  
                let html = ''  
                dados.rotas.forEach(rota => {  
                    html += `  
                        <div class="rota">  
                            <strong>${rota.nome}</strong><br>  
                            Prazo: ${rota.prazo} dias<br>  
                            Preço: ${rota.preco} ${rota.moeda}  
                        </div>  
                    `  
                })  
                  
                document.getElementById('rotas').innerHTML = html  
                document.getElementById('resultado').style.display = 'block'  
                  
            } catch (erro) {  
                document.getElementById('carregando').style.display = 'none'  
                document.getElementById('erro').innerHTML = 'Erro ao conectar: ' + erro.message  
            } finally {  
                document.getElementById('botao').disabled = false  
            }  
        }  
    </script>  
</body>  
</html>  
