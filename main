#define HALL_PIN     33
#define NUM_COLS     23
#define ACTIVE_FRAC  0.4f

const uint8_t LED_PINS[8] = {26, 14, 13, 15, 4, 17, 18, 21};

const uint8_t CHAR_V[5] = {0x07, 0x38, 0xC0, 0x38, 0x07};
const uint8_t CHAR_O[5] = {0x7E, 0x81, 0x81, 0x81, 0x7E};
const uint8_t CHAR_L[5] = {0xFF, 0x80, 0x80, 0x80, 0x80};
const uint8_t CHAR_T[5] = {0x01, 0x01, 0xFF, 0x01, 0x01};

uint8_t columnReg = 0x00;

volatile uint32_t lastRisingUs = 0;
volatile uint32_t revolutionUs = 0;
volatile bool     newPeriod    = false;

void IRAM_ATTR onHallRising() 
{
    uint32_t now = micros();
    if (lastRisingUs != 0) 
	{
        uint32_t period = now - lastRisingUs;
        if (period > 10000UL && period < 2000000UL) 
		{
            revolutionUs = period;
            newPeriod    = true;
        }
    }
    lastRisingUs = now;
}

inline uint8_t getColumn(uint8_t col) 
{
    if (col < 5)   return CHAR_V[col];
    if (col == 5)  return 0x00;
    if (col < 11)  return CHAR_O[col - 6];
    if (col == 11) return 0x00;
    if (col < 17)  return CHAR_L[col - 12];
    if (col == 17) return 0x00;
    return CHAR_T[col - 18];
}

inline void writeLEDs(uint8_t reg) 
{
    for (uint8_t bit = 0; bit < 8; bit++) 
	{
        digitalWrite(LED_PINS[bit], (reg >> bit) & 0x01);
    }
}

inline void ledsOff() 
{
    columnReg = 0x00;
    for (uint8_t i = 0; i < 8; i++) digitalWrite(LED_PINS[i], LOW);
}

void setup() {
    for (uint8_t i = 0; i < 8; i++) 
	{
        pinMode(LED_PINS[i], OUTPUT);
        digitalWrite(LED_PINS[i], LOW);
    }
    pinMode(HALL_PIN, INPUT);
    attachInterrupt(digitalPinToInterrupt(HALL_PIN), onHallRising, RISING);
    Serial.begin(115200);
    Serial.println("PoV display ready.");
}

void loop() 
{
    while (digitalRead(HALL_PIN) == LOW) { /* spin */ }

    uint32_t periodUs;
    portDISABLE_INTERRUPTS();
    periodUs = revolutionUs;
    portENABLE_INTERRUPTS();

    if (periodUs == 0) 
	{
        while (digitalRead(HALL_PIN) == HIGH) { /* spin */ }
        return;
    }

    uint32_t colDelayUs = (uint32_t)((float)periodUs * ACTIVE_FRAC / NUM_COLS);
    if (colDelayUs < 10) colDelayUs = 10;

    uint32_t windowStart = micros();

    for (uint8_t col = 0; col < NUM_COLS; col++) 
	{
        if (digitalRead(HALL_PIN) == LOW) break;

        columnReg = getColumn(col);
        writeLEDs(columnReg);

        uint32_t slotEnd = windowStart + (uint32_t)((col + 1) * colDelayUs);
        while ((int32_t)(micros() - slotEnd) < 0) { /* spin */ }
    }

    ledsOff();
    while (digitalRead(HALL_PIN) == HIGH) { /* spin */ }

    if (newPeriod) 
	{
        newPeriod = false;
        float rpm = 60000000.0f / (float)periodUs;
    }
}
