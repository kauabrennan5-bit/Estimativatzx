```javascript  
// api/frete.js - Calculadora de frete  
const BASE_URL = 'https://api.hubbuycn.com'  
  
export default async function handler(req, res) {  
  // Permite qualquer site acessar (CORS)  
  res.setHeader('Access-Control-Allow-Origin', '*')  
  res.setHeader('Access-Control-Allow-Methods', 'POST, OPTIONS')  
    
  // Se for OPTIONS, só retorna OK  
  if (req.method === 'OPTIONS') return res.status(200).end()  
    
  // Se não for POST, dá erro  
  if (req.method !== 'POST') {  
    return res.status(405).json({ erro: 'Use POST' })  
  }  
  
  try {  
    // Pega os dados que vieram do formulário  
    const { peso, comprimento, largura, altura, tipoProduto = 1 } = req.body  
      
    // Verifica se veio tudo  
    if (!peso || !comprimento || !largura || !altura) {  
      return res.status(400).json({ erro: 'Faltou peso ou medidas' })  
    }  
  
    // Pega o token de acesso (como se fosse um crachá)  
    const token = await pegarToken()  
      
    // Chama a API da HubbuyCN  
    const resposta = await fetch(`${BASE_URL}/api/Delivery/calculateDeliveryFee`, {  
      method: 'POST',  
      headers: {  
        'Content-Type': 'application/json',  
        'Authorization': `Bearer ${token}`  
      },  
      body: JSON.stringify({  
        countryId: 30,           // 30 = Brasil  
        weight: Number(peso),  
        length: Number(comprimento),  
        width: Number(largura),  
        height: Number(altura),  
        productTypeId: Number(tipoProduto)  
      })  
    })  
  
    const dados = await resposta.json()  
      
    // Verifica se deu erro de token (401 = não autorizado)  
    if (resposta.status === 401) {  
      return handler(req, res) // Tenta de novo  
    }  
  
    // Pega as rotas que vieram  
    const rotas = dados.data || []  
      
    // Formata bonitinho para o site  
    const rotasFormatadas = rotas.map(rota => ({  
      id: rota.deliveryId,  
      nome: rota.deliveryName,  
      prazo: rota.deliveryTime,  
      preco: rota.totalPrice,  
      moeda: 'CNY', // Yuan chinês  
    }))  
  
    // Manda de volta para o site  
    return res.status(200).json({  
      sucesso: true,  
      rotas: rotasFormatadas  
    })  
  
  } catch (erro) {  
    // Se deu qualquer erro, mostra  
    return res.status(500).json({ erro: erro.message })  
  }  
}  
  
// Função para pegar o token (faz login)  
let tokenGuardado = null  
let tokenExpira = 0  
  
async function pegarToken() {  
  // Se já tem token e não expirou, usa ele  
  if (tokenGuardado && Date.now() < tokenExpira) {  
    return tokenGuardado  
  }  
  
  // Pega email e senha das configurações (vamos colocar depois)  
  const email = process.env.HUBBUYCN_EMAIL  
  const senha = process.env.HUBBUYCN_PASSWORD  
  
  // Faz login  
  const login = await fetch(`${BASE_URL}/api/user/loginByPwd`, {  
    method: 'POST',  
    headers: { 'Content-Type': 'application/json' },  
    body: JSON.stringify({ email, senha })  
  })  
  
  const dados = await login.json()  
    
  if (dados.code !== 0) throw new Error('Login falhou')  
  
  // Guarda o token por 2 horas  
  tokenGuardado = dados.token  
  tokenExpira = Date.now() + (2 * 60 * 60 * 1000)  
  
  return dados.token  
}  
```  
