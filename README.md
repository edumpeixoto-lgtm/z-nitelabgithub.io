<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calculadora Zênite Lab - Soluções 3D</title>
    <style>
        :root {
            --primary: #2563eb;
            --secondary: #0f172a;
            --accent: #38bdf8;
            --background: #f8fafc;
            --card: #ffffff;
            --text: #1e293b;
            --success: #22c55e;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--background);
            color: var(--text);
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
        }

        .container {
            max-width: 900px;
            width: 100%;
        }

        header {
            text-align: center;
            margin-bottom: 30px;
            background: var(--secondary);
            color: white;
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
        }

        header h1 { margin: 0; font-size: 24px; letter-spacing: 1px; }
        header p { margin: 5px 0 0; opacity: 0.8; font-size: 14px; }

        .grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
        }

        .card {
            background: var(--card);
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05);
        }

        h2 { font-size: 18px; margin-top: 0; border-bottom: 2px solid var(--background); padding-bottom: 10px; color: var(--primary); }

        .input-group { margin-bottom: 15px; }
        label { display: block; font-size: 13px; font-weight: 600; margin-bottom: 5px; }
        input {
            width: 100%;
            padding: 10px;
            border: 1px solid #e2e8f0;
            border-radius: 6px;
            box-sizing: border-box;
            font-size: 14px;
        }

        .results {
            background: var(--secondary);
            color: white;
            grid-column: 1 / -1;
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 20px;
            padding: 25px;
        }

        .res-item { text-align: center; }
        .res-item span { display: block; font-size: 12px; opacity: 0.7; text-transform: uppercase; }
        .res-item strong { font-size: 22px; color: var(--accent); }
        .res-item.highlight strong { color: var(--success); font-size: 28px; }

        .breakdown {
            margin-top: 20px;
            font-size: 13px;
            border-top: 1px solid #475569;
            padding-top: 15px;
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
        }
    </style>
</head>
<body>

<div class="container">
    <header>
        <h1>ZÊNITE LAB</h1>
        <p>Soluções em Impressão 3D de Alta Precisão</p>
    </header>

    <div class="grid">
        <div class="card">
            <h2>Configurações Base</h2>
            <div class="input-group">
                <label>Preço do Filamento (R$/kg)</label>
                <input type="number" id="filPrice" value="120" oninput="calculate()">
            </div>
            <div class="input-group">
                <label>Custo Energia (R$/kWh)</label>
                <input type="number" id="nrgyPrice" value="0.90" step="0.01" oninput="calculate()">
            </div>
            <div class="input-group">
                <label>Potência Máquina (Watts)</label>
                <input type="number" id="power" value="350" oninput="calculate()">
            </div>
            <div class="input-group">
                <label>Mão de Obra (R$/hora)</label>
                <input type="number" id="laborRate" value="50" oninput="calculate()">
            </div>
        </div>

        <div class="card">
            <h2>Dados do Projeto</h2>
            <div class="input-group">
                <label>Peso da Peça (g)</label>
                <input type="number" id="weight" value="250" oninput="calculate()">
            </div>
            <div class="input-group">
                <label>Tempo de Impressão (horas)</label>
                <input type="number" id="printTime" value="8" oninput="calculate()">
            </div>
            <div class="input-group">
                <label>Tempo Setup/Modelagem (horas)</label>
                <input type="number" id="setupTime" value="1" oninput="calculate()">
            </div>
            <div class="input-group">
                <label>Taxa Marketplace (%)</label>
                <input type="number" id="tax" value="16.5" oninput="calculate()">
            </div>
        </div>

        <div class="card results">
            <div class="res-item">
                <span>Custo de Produção</span>
                <strong id="totalCost">R$ 0,00</strong>
            </div>
            <div class="res-item highlight">
                <span>Preço de Venda Sugerido</span>
                <strong id="sellPrice">R$ 0,00</strong>
            </div>
            <div class="res-item">
                <span>Lucro Líquido Real</span>
                <strong id="netProfit" style="color: #4ade80;">R$ 0,00</strong>
            </div>

            <div class="breakdown" id="breakdown">
                </div>
        </div>
    </div>
</div>

<script>
function calculate() {
    // Inputs
    const fPrice = parseFloat(document.getElementById('filPrice').value) || 0;
    const nPrice = parseFloat(document.getElementById('nrgyPrice').value) || 0;
    const power = parseFloat(document.getElementById('power').value) || 0;
    const lRate = parseFloat(document.getElementById('laborRate').value) || 0;
    const weight = parseFloat(document.getElementById('weight').value) || 0;
    const pTime = parseFloat(document.getElementById('printTime').value) || 0;
    const sTime = parseFloat(document.getElementById('setupTime').value) || 0;
    const tax = parseFloat(document.getElementById('tax').value) || 0;

    // Constantes de negócio
    const machinePrice = 3500;
    const machineLife = 5000;
    const packaging = 5;
    const margin = 0.30; // 30% de lucro desejado
    const failRate = 1.10; // 10% de margem de erro

    // Cálculos Individuais
    const matCost = (weight / 1000) * fPrice;
    const nrgCost = (power / 1000) * pTime * nPrice;
    const depCost = (machinePrice / machineLife) * pTime;
    const labCost = sTime * lRate;
    
    // Somas
    const baseProduction = (matCost + nrgCost + depCost + labCost) * failRate;
    const totalProduction = baseProduction + packaging;

    // Preço de Venda (Fórmula para garantir a margem após impostos/taxas)
    // Venda = (Custo / (1 - Margem - Taxa))
    const sellPrice = totalProduction / (1 - margin - (tax/100));
    const profit = sellPrice * margin;

    // Update UI
    document.getElementById('totalCost').innerText = `R$ ${totalProduction.toFixed(2)}`;
    document.getElementById('sellPrice').innerText = `R$ ${sellPrice.toFixed(2)}`;
    document.getElementById('netProfit').innerText = `R$ ${profit.toFixed(2)}`;

    document.getElementById('breakdown').innerHTML = `
        <div>Material: R$ ${matCost.toFixed(2)}</div>
        <div>Energia: R$ ${nrgCost.toFixed(2)}</div>
        <div>Depreciação: R$ ${depCost.toFixed(2)}</div>
        <div>Mão de Obra: R$ ${labCost.toFixed(2)}</div>
    `;
}

// Inicializa o cálculo
calculate();
</script>

</body>
</html># z-nitelabgithub.io
