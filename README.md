// Parâmetros do robô
input int rsiPeriod = 14;
input double rsiOverbought = 70;
input double rsiOversold = 30;
input double lotSize = 1.0; // Tamanho da posição

// Variáveis globais
datetime lastOrderTime = 0;
double lastProfit = 0;

// Função principal
void OnTick() {
    // Verificar o tempo entre as operações
    if (TimeCurrent() - lastOrderTime < 1) return;

    double rsiValue = iRSI(NULL, 0, rsiPeriod, PRICE_CLOSE, 0);
    
    // Lógica de compra
    if (rsiValue < rsiOversold) {
        if (OrderSend(Symbol(), OP_BUY, lotSize, Ask, 2, 0, 0, "RSI Buy", 0, 0, clrGreen) > 0) {
            lastOrderTime = TimeCurrent();
        }
    }
    // Lógica de venda
    else if (rsiValue > rsiOverbought) {
        if (OrderSend(Symbol(), OP_SELL, lotSize, Bid, 2, 0, 0, "RSI Sell", 0, 0, clrRed) > 0) {
            lastOrderTime = TimeCurrent();
        }
    }
}

// Função para gerenciar risco
void ManageRisk() {
    if (OrdersTotal() >= 3 || (TimeCurrent() - lastOrderTime >= 43200)) {
        for (int i = OrdersTotal() - 1; i >= 0; i--) {
            if (OrderSelect(i, SELECT_BY_POS)) {
                OrderClose(OrderTicket());
            }
        }
        Sleep(300000); // Espera 5 minutos antes de reiniciar
    }
}

// Função chamada periodicamente
void OnTimer() {
    ManageRisk();
}
