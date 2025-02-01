import cv2
import numpy as np
import talib as ta

# Função para processar a imagem do gráfico
def processar_imagem(caminho_imagem):
    # Carregar a imagem
    imagem = cv2.imread(caminho_imagem, cv2.IMREAD_GRAYSCALE)
    
    # Aplicar limiarização para binarizar a imagem
    _, imagem_binaria = cv2.threshold(imagem, 128, 255, cv2.THRESH_BINARY_INV)
    
    # Encontrar contornos na imagem
    contornos, _ = cv2.findContours(imagem_binaria, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    
    # Supondo que o gráfico seja o maior contorno
    maior_contorno = max(contornos, key=cv2.contourArea)
    
    # Extrair coordenadas do gráfico
    x, y, w, h = cv2.boundingRect(maior_contorno)
    
    # Extrair os preços (supondo que o eixo y represente os preços)
    precos = np.linspace(imagem.shape[0] - y, imagem.shape[0] - (y + h), h)
    
    return precos

# Função para calcular indicadores técnicos
def calcular_indicadores(precos):
    # Calcular Média Móvel Simples (SMA)
    sma = ta.SMA(precos, timeperiod=14)
    
    # Calcular Índice de Força Relativa (RSI)
    rsi = ta.RSI(precos, timeperiod=14)
    
    return sma, rsi

# Função para determinar pontos de compra e venda
def determinar_pontos_compra_venda(precos, sma, rsi):
    pontos_compra = []
    pontos_venda = []
    
    for i in range(1, len(precos)):
        if precos[i] > sma[i] and rsi[i] < 30:
            pontos_compra.append(i)
        elif precos[i] < sma[i] and rsi[i] > 70:
            pontos_venda.append(i)
    
    return pontos_compra, pontos_venda

# Função principal
def main(caminho_imagem):
    # Processar a imagem e extrair os preços
    precos = processar_imagem(caminho_imagem)
    
    # Calcular indicadores técnicos
    sma, rsi = calcular_indicadores(precos)
    
    # Determinar pontos de compra e venda
    pontos_compra, pontos_venda = determinar_pontos_compra_venda(precos, sma, rsi)
    
    # Exibir resultados
    print("Pontos de Compra:", pontos_compra)
    print("Pontos de Venda:", pontos_venda)

# Executar o código
if __name__ == "__main__":
    caminho_imagem = "caminho/para/sua/imagem.png"
    main(caminho_imagem)
