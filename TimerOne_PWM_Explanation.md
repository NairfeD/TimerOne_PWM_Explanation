2.  Pahami library timer dari https://github.com/ PaulStoffregen/TimerOne lalu tuliskan penjelasan Anda mengenai blok kode pada bagian       library yang mengatur duty cycle dan periode PWM. 
    Misal, penjelasan batas atas dan bawah frekuensi yang bisa di-set jika menggunakan Arduino Uno.
    Jawabannya:
    Mengenai blok kode yang mengatur duty cycle atau pwm, 
    Yang saya pahami dari library TimerOne ini dia punya beberapa fungsi yang mengatur PWM.
    1. void setPwmDuty(char pin, unsigned int duty) __attribute__((always_inline)) {...}
       Kegunaan :  
        1.a Untuk menset salah satu timer PWM pin yang akan digunakan,
	    yang juga akan menentukan register yang akan digunakan berdasarkan pin yang diset.
            Apakah TIMER1_A_PIN yang akan menjalankan OCR1A, atau TIMER1_B_PIN untuk OCR1B, 
	    atau TIMER1_C_PIN untuk OCR1C. Tapi kondisi nested-if TIMER1_B_PIN atau TIMER1_C_PIN
	    hanya akan aktif jika define TIMER1_B_PIN atatu TIMER1_C_PIN.
	    Refer block codenya:
		if (pin == TIMER1_A_PIN) OCR1A = dutyCycle;
		#ifdef TIMER1_B_PIN
		else if (pin == TIMER1_B_PIN) OCR1B = dutyCycle;   
		#endif   
		#ifdef TIMER1_C_PIN   
		else if (pin == TIMER1_C_PIN) OCR1C = dutyCycle;   
		#endif
			
	1.b Untuk menset Duty cycle yang akan dijalankan oleh register yang diset sesuai pinnya.
	    Range untuk menset Duty Cycle nya adalah 0 - 1023, dimana jika nilainya diset 0 
	    maka dutynya akan selalu LOW/"__" dan jika nilainya di set 1023 maka dutynya 
	    akan selalu HIGH/"--".
	    Refer block codenya:
		unsigned long dutyCycle = pwmPeriod;  // variable dutyCycle diinisialisasi berdasarkan variable pwmPeriod
		dutyCycle *= duty;                    // dimana pwmPeriod sendiri bertype data unsigned bisa dibilang diwali dari 0
		dutyCycle >>= 10;                     // maksimum value dari dutyCycle yang bertype unsigned long di shifting  
				                      // dengan nilainya sendiri kekanan hinga 10 bit/ memang dibatasi 10 bit(0000 0011 1111 1111 = 1023)
	        ======================================
		private:
		// properties
		static unsigned short pwmPeriod;
		static unsigned char clockSelectBits;
		======================================
   
   2. void pwm(char pin, unsigned int duty) __attribute__((always_inline)) {...}
      Kegunan:
	2.a Hampir sama seperti fungsi setPwmDuty hanya saja fungsi ini juga digunakan untuk mengeset pin output pwm dan duty nya.
	    Hanya saja ada tambahan proses yaitu menset register TCCR1A timer clock counter. TCCR1A di OR dengan nilainya sendiri sehingga 
	    outpunya sama seperti COM1A1, atau COM1B1, atatu COM1C1.
	    Refer block codenya:
		if (pin == TIMER1_A_PIN) { pinMode(TIMER1_A_PIN, OUTPUT); TCCR1A |= _BV(COM1A1); }
		#ifdef TIMER1_B_PIN  
		else if (pin == TIMER1_B_PIN) { pinMode(TIMER1_B_PIN, OUTPUT); TCCR1A |= _BV(COM1B1); }  
		#endif  
		#ifdef TIMER1_C_PIN  
		else if (pin == TIMER1_C_PIN) { pinMode(TIMER1_C_PIN, OUTPUT); TCCR1A |= _BV(COM1C1); }  
		#endif
	2.b Input Pin dan Duty pada fungsi ini diproses juga di fungsi setPwmDuty().
            Karena fungsi ini memanggil fungsi setPwmDuty.
	    Refer block codenya:
		setPwmDuty(pin, duty);  
			
   2.1. void pwm(char pin, unsigned int duty, unsigned long microseconds) __attribute__((always_inline)) {...}
        Kegunaan :
        Ada tambahan 1 input input pada fungsi ini yaitu pada variable "microseconds" atau kita bisa mengeset periode pwm yang kita set dalam microseconds.
	Dan fungsi ini juga memanggil/merefer ke fungsi pwm dengan 2 inpunt.
	Refer block codenya:	
		if (microseconds > 0) setPeriod(microseconds);
		pwm(pin, duty);
			
   3. void disablePwm(char pin) __attribute__((always_inline)) {...} 
      Kegunaan : 
        Menstop/mendisable penggunaan PWM pada pin yang diinput. Dengan cara meng AND kan TCCR1A dengan nilainya sendiri sehingga
	outputnya sama dengan negasi COM1A1, atau COM1B1, atatu COM1C1.
      Refer block codenya:
		if (pin == TIMER1_A_PIN) TCCR1A &= ~_BV(COM1A1);
		#ifdef TIMER1_B_PIN  
		else if (pin == TIMER1_B_PIN) TCCR1A &= ~_BV(COM1B1);  
		#endif  
		#ifdef TIMER1_C_PIN  
		else if (pin == TIMER1_C_PIN) TCCR1A &= ~_BV(COM1C1);  
		#endif
