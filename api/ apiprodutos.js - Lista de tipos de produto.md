// api/produtos.js - Lista de tipos de produto  
const BASE_URL = 'https://api.hubbuycn.com'  
  
export default async function handler(req, res) {  
  res.setHeader('Access-Control-Allow-Origin', '*')  
    
  try {  
    const email = process.env.HUBBUYCN_EMAIL  
    const senha = process.env.HUBBUYCN_PASSWORD  
  
    // Faz login  
    const login = await fetch(`${BASE_URL}/api/user/loginByPwd`, {  
      method: 'POST',  
      headers: { 'Content-Type': 'application/json' },  
      body: JSON.stringify({ email, senha })  
    })  
      
    const { token } = await login.json()  
  
    // Pega os tipos de produto  
    const resposta = await fetch(`${BASE_URL}/api/Product/getProductTypes`, {  
      headers: { 'Authorization': `Bearer ${token}` }  
    })  
      
    const dados = await resposta.json()  
      
    // Formata bonitinho  
    const tipos = (dados.data || []).map(t => ({  
      id: t.productTypeId,  
      nome: t.productTypeName  
    }))  
      
    res.status(200).json({ tipos })  
      
  } catch (erro) {  
    res.status(500).json({ erro: erro.message })  
  }  
}  
