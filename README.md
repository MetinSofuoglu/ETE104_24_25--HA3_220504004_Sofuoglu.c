#include <msp430.h>

// Timer tabanlı gecikme fonksiyonu (yaklaşık 0.5 saniye)
void delay_timerB() {
    TB0CTL = TBSSEL_2 + MC_1 + ID_3; // SMCLK, Up mode, input divider /8
    TB0CCR0 = 62500;                 // 0.5 saniye için değer (saat frekansına bağlı)
    TB0CCTL0 = CCIE;                 // Kesmeyi etkinleştir
    __bis_SR_register(LPM0_bits + GIE); // Low power mode + kesmeleri etkinleştir
}

// Timer kesme servisi
#pragma vector = TIMER0_B0_VECTOR
__interrupt void Timer_B (void) {
    TB0CCTL0 &= ~CCIE;                  // Kesme devre dışı
    __bic_SR_register_on_exit(LPM0_bits); // Low power moddan çık
}

void main(void) {
    WDTCTL = WDTPW | WDTHOLD;       // Watchdog timer durdur
    P1DIR |= BIT0;                  // P1.0 çıkış (LED2)
    P1DIR &= ~(BIT1 | BIT2);        // P1.1 (S1) ve P1.2 (S2) giriş
    P1REN |= BIT1 | BIT2;           // Pull-up/down dirençleri etkin
    P1OUT |= BIT1 | BIT2;           // Pull-up dirençler

    while (1) {
        if (!(P1IN & BIT1)) {       // S1'e basılmışsa
            P1OUT ^= BIT0;          // LED2'yi değiştir
            delay_timerB();         // Gecikme
        }

        if (!(P1IN & BIT2)) {       // S2'ye basılmışsa (ek kontrol)
            P1OUT &= ~BIT0;         // LED2'yi kapat
        }
    }
}
