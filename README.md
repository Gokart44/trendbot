# class OverkillScalperMAX extends UserDefinedIndicator {
    onInit() {
        return {
            title: "Overkill Scalper MAX v2.0",
            shortTitle: "OS MAX v2",
            description: "Enhanced liquidity + FVG + multi-TF scalper based on Overkill Scalper"
        };
    }

    onCalculate(data) {
        const close = data.close;
        const high = data.high;
        const low = data.low;
        const volume = data.volume;
        const open = data.open;
        const length = data.length;

        const rsi = this.rsi(close, 14);
        const emaFast = this.ema(close, 8);
        const emaSlow = this.ema(close, 21);
        const liqHigh = this.highest(high, 25);
        const liqLow = this.lowest(low, 25);
        const volAvg = this.sma(volume, 20);

        const fvgBull = [];
        const fvgBear = [];
        for (let i = 2; i < length; i++) {
            fvgBull[i] = (low[i-2] > high[i]) ? (low[i-2] + high[i]) / 2 : NaN;
            fvgBear[i] = (high[i-2] < low[i]) ? (high[i-2] + low[i]) / 2 : NaN;
        }

        const higherTFTrend = this.security("15", () => close > this.ema(close, 50));

        const buySignal = [];
        const sellSignal = [];

        for (let i = 0; i < length; i++) {
            buySignal[i] = (rsi[i] < 32) &&
                           (close[i] > emaFast[i]) &&
                           (close[i] > emaSlow[i]) &&
                           (close[i] > liqLow[i]) &&
                           (close[i-1] <= liqLow[i-1]) &&
                           (volume[i] > volAvg[i] * 1.8) &&
                           (!isNaN(fvgBull[i]) && close[i] > fvgBull[i]) &&
                           (higherTFTrend[i] === 1);

            sellSignal[i] = (rsi[i] > 68) &&
                            (close[i] < emaFast[i]) &&
                            (close[i] < emaSlow[i]) &&
                            (close[i] < liqHigh[i]) &&
                            (close[i-1] >= liqHigh[i-1]) &&
                            (volume[i] > volAvg[i] * 1.8) &&
                            (!isNaN(fvgBear[i]) && close[i] < fvgBear[i]) &&
                            (higherTFTrend[i] === -1);
        }

        this.plotHistogram(buySignal, "MAX LONG", "#00ff00", 3);
        this.plotHistogram(sellSignal, "MAX SHORT", "#ff0000", 3);

        this.plot(emaFast, "Fast EMA", "#ffff00");
        this.plot(emaSlow, "Slow EMA", "#ffffff");
    }
}
