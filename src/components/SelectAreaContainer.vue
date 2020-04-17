<template>
  <div id="select-container" class="select-area" v-on:mousedown="onMouseDown">
    <slot/>
    <div class="select-box" v-if="mouseDown" :style="boxStyle">
      <div class="resizable_borders">
        <div class="row_top">
          <div class="corner_top-left" v-on:mousedown="onDiagonalResize"></div>
          <div class="top" v-on:mousedown="onVerticalResize"></div>
          <div class="corner_top-right" v-on:mousedown="onDiagonalResize"></div>
        </div>
        <div class="row_middle">
          <div class="left" v-on:mousedown="onHorizontalResize"></div>
          <div class="inner" v-on:mousedown="onMove"></div>
          <div class="right" v-on:mousedown="onHorizontalResize"></div>
        </div>
        <div class="row_bottom">
          <div class="corner_bottom-left" v-on:mousedown="onDiagonalResize"></div>
          <div class="bottom" v-on:mousedown="onVerticalResize"></div>
          <div class="corner_bottom-right" v-on:mousedown="onDiagonalResize"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'SelectAreaContainer',
    data() {
      return {
        startPoint: null,
        endPoint: null,
        mouseDown: false,
        bounds: {},
        moveStartPoint: null,
      };
    },
    methods: {
      onMouseDown(event) {
        event.stopPropagation();

        if (this.mouseDown) return this.reset();
        if (event.target.parentElement.id !== 'select-container') return;

        // Ignore right clicks
        if (event.button === 2) return;

        this.startPoint = { x: event.pageX, y: event.pageY };
        window.addEventListener('mousemove', this.onMouseMove);
        window.addEventListener('mouseup', this.onMouseUp);
      },
      onMouseMove({ pageX, pageY }) {
        this.mouseDown = true;
        this.endPoint = { x: this.getX(pageX), y: this.getY(pageY) };
      },
      onMouseUp() {
        window.removeEventListener('mousemove', this.onMouseMove);
        window.removeEventListener('mouseup', this.onMouseUp);
      },
      onVerticalResize(event) {
        event.stopPropagation();
        window.addEventListener('mousemove', this.handleVerticalResize);
        window.addEventListener('mouseup', this.stopVerticalResize);
      },
      handleVerticalResize({ pageY }) {
        const y = this.getY(pageY);
        const point = this.getPointToChangeY(y);
        point.y = y;
      },
      stopVerticalResize() {
        window.removeEventListener('mousemove', this.handleVerticalResize);
        window.removeEventListener('mouseup', this.stopVerticalResize);
      },
      onHorizontalResize(event) {
        event.stopPropagation();
        window.addEventListener('mousemove', this.handleHorizontalResize);
        window.addEventListener('mouseup', this.stopHorizontalResize);
      },
      handleHorizontalResize({ pageX }) {
        const x = this.getX(pageX);
        const point = this.getPointToChangeX(x);
        point.x = x;
      },
      stopHorizontalResize() {
        window.removeEventListener('mousemove', this.handleHorizontalResize);
        window.removeEventListener('mouseup', this.stopHorizontalResize);
      },
      onDiagonalResize(event) {
        event.stopPropagation();
        window.addEventListener('mousemove', this.handleDiagonalResize);
        window.addEventListener('mouseup', this.stopDiagonalResize);
      },
      handleDiagonalResize({ pageX, pageY }) {
        const x = this.getX(pageX);
        const y = this.getY(pageY);
        const pointToChangeX = this.getPointToChangeX(x);
        const pointToChangeY = this.getPointToChangeY(y);

        pointToChangeX.x = x;
        pointToChangeY.y = y;
      },
      stopDiagonalResize() {
        window.removeEventListener('mousemove', this.handleDiagonalResize);
        window.removeEventListener('mouseup', this.stopDiagonalResize);
      },
      onMove(event) {
        event.stopPropagation();
        this.moveStartPoint = { x: event.pageX, y: event.pageY };
        window.addEventListener('mousemove', this.handleMove);
        window.addEventListener('mouseup', this.stopMove);
      },
      handleMove({ pageX, pageY }) {
        const { left, right, top, bottom } = this.bounds;
        const movedByX = pageX - this.moveStartPoint.x;
        const movedByY = pageY - this.moveStartPoint.y;
        const startX = this.startPoint.x + movedByX;
        const startY = this.startPoint.y + movedByY;
        const endX = this.endPoint.x + movedByX;
        const endY = this.endPoint.y + movedByY;
        const startXInBounds = left <= startX && startX <= right;
        const endXInBounds = left <= endX && endX <= right;
        const startYInBounds = top <= startY && startY <= bottom;
        const endYInBounds = top <= endY && endY <= bottom;

        if (startXInBounds && endXInBounds) {
          this.startPoint.x = startX;
          this.endPoint.x = endX;
        }
        if (startYInBounds && endYInBounds) {
          this.startPoint.y = startY;
          this.endPoint.y = endY;
        }
        this.moveStartPoint = { x: pageX, y: pageY };
      },
      stopMove() {
        window.removeEventListener('mousemove', this.handleMove);
        window.removeEventListener('mouseup', this.stopMove);
      },
      reset() {
        this.startPoint = null;
        this.endPoint = null;
        this.mouseDown = false;
      },
      getPointToChangeX(x) {
        const startDeltaX = Math.abs(x - this.startPoint.x);
        const endDeltaX = Math.abs(x - this.endPoint.x);

       return (startDeltaX < endDeltaX) ? this.startPoint : this.endPoint;
      },
      getPointToChangeY(y) {
        const startDeltaY = Math.abs(y - this.startPoint.y);
        const endDeltaY = Math.abs(y - this.endPoint.y);

        return (startDeltaY < endDeltaY) ? this.startPoint : this.endPoint;
      },
      getX(x) {
        return (x > this.bounds.right)
          ? this.bounds.right
          : Math.max(this.bounds.left, x);
      },
      getY(y) {
        return (y > this.bounds.bottom)
          ? this.bounds.bottom
          : Math.max(this.bounds.top, y);
      },
    },
    computed: {
      selectionBox() {
        if (!this.mouseDown || !this.startPoint || !this.endPoint) return {};

        const left = Math.min(this.startPoint.x, this.endPoint.x);
        const top = Math.min(this.startPoint.y, this.endPoint.y);
        const width = Math.abs(this.startPoint.x - this.endPoint.x);
        const height = Math.abs(this.startPoint.y - this.endPoint.y);

        return { left, top, width, height };
      },
      boxStyle() {
        if (!this.mouseDown || !this.startPoint || !this.endPoint) {
          return {};
        }
        const { left, top, width, height } = this.selectionBox;

        return {
          left: `${left}px`,
          top: `${top}px`,
          width: `${width}px`,
          height: `${height}px`,
        };
      },
    },
    mounted() {
      const { top, bottom, left, right } = this.$el.getBoundingClientRect();
      this.bounds = { top, bottom, left, right };
    },
    beforeDestroy() {
      window.removeEventListener('mousemove', this.onMouseMove);
      window.removeEventListener('mouseup', this.onMouseUp);
      this.$children.forEach(child => child.$off('click'));
    },
  };
</script>

<style scoped>
  .select-area {
    width: fit-content;
  }

  .select-area .select-box {
    position: absolute;
    z-index: 1;
    border: 1px dashed black;
    width: 100%;
    height: 100%;
  }

  .resizable_borders {
    display: flex;
    flex-direction: column;
    width: 100%;
    height: 100%;
  }

  .row_top, .row_bottom {
    display: flex;
    height: 2px;
    width: 100%;
  }

  .row_middle {
    display: flex;
    height: calc(100% - 4px);
    width: 100%;
  }

  .corner_top-left, .corner_top-right, .corner_bottom-right, .corner_bottom-left {
    height: 100%;
    width: 2px;
    background-color: black;
    opacity: 0.2;
    /*background-color: red;*/
  }

  .corner_top-left, .corner_bottom-right {
    cursor: nwse-resize;
  }

  .corner_top-right, .corner_bottom-left {
    cursor: nesw-resize;
  }

  .left, .right {
    height: 100%;
    width: 2px;
    cursor: col-resize;
    background-color: black;
    opacity: 0.2;
    /*background-color: blue;*/
  }

  .top, .bottom {
    height: 100%;
    width: calc(100% - 4px);
    cursor: row-resize;
    background-color: black;
    opacity: 0.2;
    /*background-color: blue;*/
  }

  .inner {
    height: 100%;
    width: calc(100% - 4px);
    background-color: black;
    opacity: 0.2;
    cursor: pointer;
  }
</style>
