<template>
  <div :style="{ height: size + 'px', width: size + 'px' }">
    <canvas ref="roseCanvas" style="position: absolute; left: 0; top: 0;" />
    <canvas ref="odometerCanvas" style="position: absolute;" />
  </div>
</template>

<script>
import {
  drawBackground,
  drawFrame,
  drawForeground,
  BackgroundColor,
  FrameDesign,
  ForegroundType,
  Odometer,
} from "steelseries";

import { toNumber, toArray, toBoolean, toUpper } from "./util";

import Rose from "./RGraph.rose";

/**
 * Wind Rose
 * @displayName Wind Rose
 */
export default {
  name: "WindRose",
  props: {
    /**
     * Background Color of Dial
     * @values DARK_GRAY, SATIN_GRAY, LIGHT_GRAY, WHITE, BLACK, BEIGE, BROWN, RED, GREEN, BLUE,
     * TURNED, ANTHRACITE, MUD, PUNCHED_SHEET, CARBON, STAINLESS, BRUSHED_METAL, BRUSHED_STAINLESS
     */
    backgroundColor: {
      default: "DARK_GRAY",
      required: false,
      type: String,
      validator: (value) => value.toUpperCase() in BackgroundColor,
    },
    /**
     * Display the dial background
     */
    backgroundVisible: {
      default: true,
      required: false,
      type: [Boolean, String],
    },
    /**
     * Sets the foreground styling type
     * @values TYPE1 through TYPE5
     */
    foregroundType: {
      default: "TYPE1",
      required: false,
      type: String,
      validator: (value) => value.toUpperCase() in ForegroundType,
    },
    /**
     * Display the foreground style (from foregroundType) on the dial
     */
    foregroundVisible: {
      default: true,
      required: false,
      type: [Boolean, String],
    },
    /**
     * Select the frame design style used on the dial
     * @values BLACK_METAL, METAL, SHINY_METAL, BRASS, STEEL, CHROME, GOLD, ANTHRACITE,
     *         TILTED_GRAY, TILTED_BLACK, GLOSSY_METAL
     */
    frameDesign: {
      default: "METAL",
      required: false,
      type: String,
      validator: (value) => value.toUpperCase() in FrameDesign,
    },
    /**
     * Display the frame (with the frameDesign selected) around the dial
     */
    frameVisible: {
      default: true,
      required: false,
      type: [Boolean, String],
    },
    /**
     * Color to be used for the graph radials
     */
    radialColor: {
      default: "Gradient(#408040:red:#7070A0)",
      required: false,
      type: String,
    },
    /**
     * Total number of digits (including decimal) to display
     */
    odometerDigits: {
      default: 5,
      required: false,
      type: [Number, String],
      validator: (value) => toNumber(value) > 0,
    },
    /**
     * Set the font to use
     */
    odometerFont: {
      default: "sans-serif",
      required: false,
      type: String,
    },
    /**
     * Set the value to display on the odometer
     */
    odometerValue: {
      default: undefined,
      required: false,
      type: [Number, String],
      validator: (value) => !Number.isNaN(value),
    },
    /**
     * Set to enable the odometer display on the rose gauge
     */
    odometerVisible: {
      default: false,
      required: false,
      type: [Boolean, String],
    },
    /**
     * HTML color of the value background
     */
    odometerBackColor: {
      default: "#050505",
      required: false,
      type: String,
    },
    /**
     * HTML color of the value foreground
     */
    odometerForeColor: {
      default: "#F8F8F8",
      required: false,
      type: String,
    },
    /**
     * Set the wobble factor of the digits, set to zero for perfect alignment
     */
    odometerWobbleFactor: {
      default: 0.07,
      required: false,
      type: [Number, String],
      validator: (value) => !Number.isNaN(value),
    },
    /**
     * Set the size in pixels of the canvas (height and width)
     */
    size: {
      default: 150,
      required: false,
      type: [Number, String],
      validator: (value) => !Number.isNaN(value),
    },
    /**
     * Set the title
     */
    title: {
      default: undefined,
      required: false,
      type: String,
    },
    /**
     * Rose array of 16 values (starting at north) to plot
     */
    value: {
      required: true,
      type: [Array, String],
    },
  },
  methods: {
    buildFrame() {
      if (!toBoolean(this.frameVisible)) return;
      const size = toNumber(this.size);
      this.buffers.frame = document.createElement("canvas");
      this.buffers.frame.width = size;
      this.buffers.frame.height = size;
      const ctx = this.buffers.frame.getContext("2d");
      drawFrame(
        ctx,
        FrameDesign[toUpper(this.frameDesign)],
        size / 2,
        size / 2,
        size,
        size
      );
    },
    buildBackground() {
      if (!toBoolean(this.backgroundVisible)) return;
      const size = toNumber(this.size);
      this.buffers.background = document.createElement("canvas");
      this.buffers.background.width = size;
      this.buffers.background.height = size;
      const ctx = this.buffers.background.getContext("2d");
      drawBackground(
        ctx,
        BackgroundColor[toUpper(this.backgroundColor)],
        size / 2,
        size / 2,
        size,
        size
      );

      const compassPoints = ["N", "E", "S", "W"];
      ctx.font = 0.08 * size + "px serif";
      ctx.fillStyle = BackgroundColor[
        toUpper(this.backgroundColor)
      ].labelColor.getRgbColor();
      ctx.textAlign = "center";
      ctx.textBaseline = "middle";
      for (var i = 0; i < 4; i++) {
        ctx.translate(size / 2, size * 0.125);
        ctx.fillText(compassPoints[i], 0, 0, size);
        ctx.translate(-size / 2, -size * 0.125);
        ctx.translate(size / 2, size / 2);
        ctx.rotate(Math.PI / 2);
        ctx.translate(-size / 2, -size / 2);
      }
    },
    buildForeground() {
      if (!toBoolean(this.foregroundVisible)) return;
      const size = toNumber(this.size);
      this.buffers.foreground = document.createElement("canvas");
      this.buffers.foreground.width = size;
      this.buffers.foreground.height = size;
      const ctx = this.buffers.background.getContext("2d");
      drawForeground(
        ctx,
        ForegroundType[toUpper(this.foregroundType)],
        size,
        size,
        false
      );
    },
    buildTitle() {
      if (!this.title) return;
      const size = toNumber(this.size);
      this.buffers.title = document.createElement("canvas");
      this.buffers.title.width = size;
      this.buffers.title.height = size;
      const ctx = this.buffers.title.getContext("2d");
      const backgroundColor = BackgroundColor[toUpper(this.backgroundColor)];
      ctx.font = 0.046728 * size + "px Arial,Verdana,sans-serif";
      ctx.textAlign = "center";
      ctx.textBaseline = "middle";
      ctx.fillStyle = backgroundColor.labelColor.getRgbColor();
      ctx.strokeStyle = backgroundColor.labelColor.getRgbaColor();
      ctx.fillStyle = backgroundColor.labelColor.getRgbaColor();
      ctx.fillText(this.title, size / 2, size * 0.3, size * 0.3);
    },
    buildPlot() {
      const size = toNumber(this.size);
      this.buffers.plot = document.createElement("canvas");
      this.buffers.plot.width = size;
      this.buffers.plot.height = size;
      new Rose({
        canvas: this.buffers.plot,
        data: toArray(this.value),
        options: {
          margin: Math.ceil(40 / this.value.length),
          colors: [this.radialColor],
          colorsAlpha: 0.5,
          colorsStroke: "black",
          axes: true,
          axesColor: "grey",
          backgroundGridRadialsCount: this.value.length,
          textAccessible: false,
          labelsAxes: "",
        },
      }).draw();
    },
    drawGraph() {
      const canvas = this.$refs["roseCanvas"];
      canvas.width = toNumber(this.size);
      canvas.height = toNumber(this.size);
      const ctx = canvas.getContext("2d");
      this.buffers.frame && ctx.drawImage(this.buffers.frame, 0, 0);
      this.buffers.background && ctx.drawImage(this.buffers.background, 0, 0);
      this.buffers.plot && ctx.drawImage(this.buffers.plot, 0, 0);
      this.buffers.foreground && ctx.drawImage(this.buffers.foreground, 0, 0);
      this.buffers.title && ctx.drawImage(this.buffers.title, 0, 0);
    },
    drawOdometer() {
      if (!toBoolean(this.odometerVisible)) return;
      const size = toNumber(this.size);
      const digits = toNumber(this.odometerDigits);
      const height = Math.ceil(size * 0.08);
      const width = Math.ceil(Math.floor(height * 0.68) * digits);
      const top = Math.ceil(size * 0.75);
      const left = Math.ceil((size - width) / 2);
      const ctx = this.$refs["odometerCanvas"];
      ctx.height = height;
      ctx.width = width;
      ctx.style.left = left + "px";
      ctx.style.top = top + "px";
      this.odometer = new Odometer(ctx, {
        digits: digits - 1,
        decimals: 1,
        font: this.odometerFont,
        valueBackColor: this.odometerBackColor,
        valueForeColor: this.odometerForeColor,
        wobbleFactor: toNumber(this.odometerWobbleFactor),
        value: toNumber(this.odometerValue),
      });
    },
    draw() {
      //this.sizeCanvas();
      this.buildFrame();
      this.buildBackground();
      this.buildForeground();
      this.buildTitle();
      this.buildPlot();
      this.drawGraph();
      this.drawOdometer();
    },
  },
  data() {
    return {
      buffers: {},
      odometer: undefined,
    };
  },
  mounted() {
    this.draw();
  },
  watch: {
    radialColor() {
      this.buildPlot();
      this.drawGraph();
    },
    size() {
      this.draw();
    },
    title(newValue) {
      this.buffers.title = newValue;
      this.drawTitle();
      this.drawGraph();
    },
    value() {
      this.buildPlot();
      this.drawGraph();
    },
    odometerValue(newValue) {
      this.odometer && this.odometer.setValueAnimated(toNumber(newValue));
    },
  },
};
</script>
