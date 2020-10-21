<template>
  <div ref="rose">
    <canvas ref="canvas" />
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
} from "steelseries";

import { toNumber, toBoolean, toUpper } from "./util";

import "./RGraph.common.core";
import "./RGraph.rose";

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
     * Color to be used for the graph radials
     */
    radialColor: {
      default: "Gradient(#408040:red:#7070A0)",
      required: false,
      type: String,
    },
    /**
     * Set the size in pixels of the canvas (height and width)
     */
    size: {
      default: undefined,
      required: false,
      type: [Number, String],
      validator: (value) => toNumber(value) > 0,
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
     * Rose array of 16 values to plot
     */
    value: {
      required: true,
      type: Array,
    },
  },
  methods: {
    drawRose() {
      const ctx = this.$refs["canvas"];
      const size = this.size
        ? toNumber(this.size)
        : Math.min(ctx.width, ctx.height);
      ctx.width = size;
      ctx.height = size;

      // steelseries gauge frame
      if (!this.buffers.frame) {
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
      }

      // Create a steelseries gauge background
      if (!this.buffers.background) {
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
        ctx.save();
        // set the font
        ctx.font = 0.08 * size + "px serif";
        ctx.fillStyle = BackgroundColor[
          toUpper(this.backgroundColor)
        ].labelColor.getRgbColor();
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        // Draw the compass points
        const compassPoints = ["N", "E", "S", "W"];
        for (var i = 0; i < 4; i++) {
          ctx.translate(size / 2, size * 0.125);
          ctx.fillText(compassPoints[i], 0, 0, size);
          ctx.translate(-size / 2, -size * 0.125);
          // Move to center
          ctx.translate(size / 2, size / 2);
          ctx.rotate(Math.PI / 2);
          ctx.translate(-size / 2, -size / 2);
        }
        ctx.restore();
      }

      // Create a steelseries gauge foreground
      if (!this.buffers.foreground) {
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
        ctx.save();
        // draw title
        if (this.title) {
          const backgroundColor =
            BackgroundColor[toUpper(this.backgroundColor)];
          ctx.font = 0.046728 * size + "px Arial,Verdana,sans-serif";
          ctx.textAlign = "center";
          ctx.textBaseline = "middle";
          ctx.fillStyle = backgroundColor.labelColor.getRgbColor();
          ctx.strokeStyle = backgroundColor.labelColor.getRgbaColor();
          ctx.fillStyle = backgroundColor.labelColor.getRgbaColor();
          ctx.font = 0.06 * size + "px serif";
          ctx.fillText(this.title, size / 2, size * 0.3, size * 0.3);
          ctx.restore();
        }
      }

      // rose plot canvas buffer
      this.buffers.plot = document.createElement("canvas");
      this.buffers.plot.width = size;
      this.buffers.plot.height = size;

      // Create a new rose plot
      const rose = new RGraph.Rose({
        canvas: this.buffers.plot,
        data: this.value,
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
      });
      rose.draw();

      const ctxRoseCanvas = ctx.getContext("2d");
      const offset = Math.floor(size / 2 - this.buffers.plot.width / 2);
      ctxRoseCanvas.drawImage(this.buffers.frame, 0, 0);
      ctxRoseCanvas.drawImage(this.buffers.background, 0, 0);
      ctxRoseCanvas.drawImage(this.buffers.plot, offset, offset);
      ctxRoseCanvas.drawImage(this.buffers.foreground, 0, 0);
    },
  },
  data() {
    return {
      buffers: {},
    };
  },
  mounted() {
    this.drawRose();
  },
  watch: {
    radialColor() {
      this.drawRose();
    },
    size() {
      this.buffers = {};
      this.drawRose();
    },
    title() {
      this.buffers.foreground = undefined;
      this.drawRose();
    },
    value() {
      this.drawRose();
    },
  },
};
</script>
