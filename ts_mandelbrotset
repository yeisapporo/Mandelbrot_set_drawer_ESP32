#include <Adafruit_GFX.h>
#include <SPI.h>
#include <Wire.h>
#include <Adafruit_I2CDevice.h>
#include <Adafruit_ILI9341.h>
#include <XPT2046_Touchscreen.h>

// タッチスクリーンのキャリブレーション用データ(個別に固定的に補正)
#define TS_MINX  (400)
#define TS_MINY  (130)
#define TS_MAXX (3800)
#define TS_MAXY (3800)

// XPT2046のCS
#define XPT2046_CS (15)

// 画面(VSPI)
#define TFT_DC    (26)
#define TFT_CS    ( 5)
#define TFT_RST   (16)

XPT2046_Touchscreen ts = XPT2046_Touchscreen(XPT2046_CS, -1);
Adafruit_ILI9341 tft = Adafruit_ILI9341(TFT_CS, TFT_DC, TFT_RST);
// タッチスクリーン(HSPI)
SPIClass hSPI(HSPI);

void setup(void) {
  Serial.begin(115200);
  
  tft.begin(40000000);
  tft.fillScreen(ILI9341_BLACK);

  if (!ts.begin(hSPI)) {
    Serial.println("Couldn't start touchscreen controller.");
    for(;;) {}
  }
  ts.setRotation(2);
  Serial.println("Touchscreen started.");
}

// 描画範囲定義
#define X_MAX (240)
#define Y_MAX (240)

double startR = -2.0;
double endR = 1.0;
double startI = -1.5;
double endI = 1.5;
// 描画バッファ(8分割)
unsigned short buff[X_MAX * Y_MAX / 8];

void drawMandelbrotSet(void) {
  double E = 4.0;
  int iMax = 256;

  double real, imag, dR, dI;
  double zR, zI, nextZR, nextZI;
  int i, x, y;
  unsigned char r, g, b;

  int buffY = 0;
  int buffCnt = 0;

  dR = (endR - startR) / X_MAX; // 実部刻み幅
  dI = (endI - startI) / Y_MAX; // 虚部刻み幅

  for(imag = startI, y = 0; y < Y_MAX; imag += dI, y++) {
    for(real = startR, x = 0; x < X_MAX; real += dR, x++) {
      zR = 0;
      zI = 0;
      for(i = 0; i < iMax; i++) {
        nextZR = zR * zR - zI * zI + real;  // 実部漸化式計算
        nextZI = 2 * zR *zI + imag;         // 虚部漸化式計算
        // 発散の判定
        if((zR *zR + zI * zI) > E) {
          break;
        }
        zR = nextZR;
        zI = nextZI;
      }
      if(i < iMax) {
        // 集合の外(発散)
        r = ((i - 24) % 24) * 24;
        g = ((i - 16) % 16) * 8;
        b = ((i) % 4) * 4;

        buff[x + X_MAX * buffY] = r * 256 + g * 64 + b;
      } else {
        // 集合の中(収束)
        buff[x + X_MAX * buffY] = 0;
      }
    }
    buffY++;
    // バッファに1ブロック分データを入れたら
    if(buffY >= (Y_MAX / 8)) {   
      // 1ブロック描画
      tft.drawRGBBitmap(0, buffCnt * Y_MAX / 8, buff, X_MAX, Y_MAX / 8);
      buffY = 0;
      buffCnt++;
    }
  }
}

void loop() {
  double dReal;
  double dImag;
  static bool first = true;

  if(first) {
      drawMandelbrotSet();
      first = false;
  }

  tft.drawRect(0, 0, X_MAX, Y_MAX, ILI9341_DARKGREY);

  // タッチ情報がなければ何もしない
  if (ts.bufferEmpty() || !ts.touched()) {
    delay(1);
    return;
  }

  // タッチ場所生座標取得  
  TS_Point p = ts.getPoint();

  // 画面座標に変換
  p.x = map(p.x, TS_MINX, TS_MAXX, 0, tft.width());
  p.y = map(p.y, TS_MINY, TS_MAXY, 0, tft.height());

  // 刻み幅を計算
  dReal = (endR - startR) / X_MAX;
  dImag = (endI - startI) / Y_MAX;

  // タッチ場所が描画の中心となるよう始点・終点を計算  
  startR -= dReal * (X_MAX / 2 - p.x);
  endR   -= dReal * (X_MAX / 2 - p.x);
  startI -= dImag * (Y_MAX / 2 - p.y);
  endI   -= dImag * (Y_MAX / 2 - p.y);

  // 拡大
  startR += dReal * 64;
  endR   -= dReal * 64;
  startI += dImag * 64;
  endI   -= dImag * 64;
  
  // 描画
  drawMandelbrotSet();
  delay(1);
}
