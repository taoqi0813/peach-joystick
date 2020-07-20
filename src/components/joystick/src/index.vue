<template>
  <div
    v-if="!defaultOptions.hidden"
    id="peach-joystick-container"
    class="peach-joystick-container"
    :class="{'peach-joystick-left-bottom': defaultOptions.position === 'left-bottom','peach-joystick-right-bottom': defaultOptions.position === 'right-bottom'}"
    :style="{width: parseInt(defaultOptions.size) + defaultOptions.padding * 2 + 'px', height: parseInt(defaultOptions.size) + defaultOptions.padding * 2 + 'px',zIndex: defaultOptions.zIndex}"
  >
    <canvas id="joystick-canvas" :width="defaultOptions.size" :height="defaultOptions.size" />
  </div>
</template>

<script>
// 全局变量
var canvas,
  el,
  joystick_cmd,
  joystick_bg,
  joystick_height,
  joystick_left,
  body_height,
  offset_height,
  offset_left,
  offset_top,
  rb_size,
  rc_size,
  centerX,
  centerY,
  context;
// 摇杆头应当移动到的位置
var jx = 0;
var jy = 0;
// 摇杆移动的方向
var moveForward = false;
var moveBackward = false;
var moveLeft = false;
var moveRight = false;
export default {
  name: "PeachJoystick",
  props: {
    options: {
      type: Object
    }
  },
  watch: {
    options: {
      handler() {
        this.initJoystick();
      },
      deep: true //true 深度监听
    }
  },
  data() {
    return {
      defaultOptions: {
        size: "120",
        hidden: false,
        background: require("../../../assets/joystick_bg.png"),
        joystick: require("../../../assets/joystick_cmd.png"),
        position: "left-bottom",
        padding: 25,
        zIndex: 100
      }
    };
  },
  mounted() {
    this.initJoystick();
  },
  beforeDestroy() {
    el.removeEventListener("touchstart", this.touch, false);
    el.removeEventListener("touchmove", this.touch, false);
    el.removeEventListener("touchend", this.touch, false);
  },
  destroyed() {
    if (this.defaultOptions.appendToBody && this.$el && this.$el.parentNode) {
      this.$el.parentNode.removeChild(this.$el);
    }
  },
  methods: {
    initJoystick() {
      this.defaultOptions = {
        ...this.defaultOptions,
        ...this.options
      };
      this.$nextTick(() => {
        if (this.defaultOptions.appendToBody) {
          document.body.appendChild(this.$el);
        }
        joystick_cmd = new Image(); // 内摇杆图片
        joystick_bg = new Image(); // 外摇杆图片
        canvas = document.getElementById("joystick-canvas"); // 画板

        el = document.getElementById("peach-joystick-container");
        el.addEventListener("touchstart", this.touch, false);
        el.addEventListener("touchmove", this.touch, false);
        el.addEventListener("touchend", this.touch, false);

        // 摇杆区高度
        joystick_height = el.clientHeight;
        joystick_left = el.offsetLeft;
        body_height = document.body.clientHeight;
        offset_height = body_height - joystick_height;
        offset_left = canvas.offsetLeft;
        offset_top = canvas.offsetTop;
        // console.log({
        //   el,
        //   joystick_height,
        //   joystick_left,
        //   body_height,
        //   offset_height,
        //   offset_left,
        //   offset_top
        // });
        rb_size = canvas.height; // 外摇杆大小
        rc_size = canvas.height * 0.6; // 内摇杆大小
        centerX = canvas.height / 2; // 摇杆中心x坐标
        centerY = canvas.height / 2; // 摇杆中心y坐标
        context = canvas.getContext("2d"); // 画布
        joystick_cmd.src = this.defaultOptions.joystick;
        joystick_bg.src = this.defaultOptions.background;

        // 图片加载完成时执行这俩函数
        joystick_cmd.onload = function() {
          context.drawImage(
            joystick_cmd,
            (rb_size - rc_size) / 2,
            (rb_size - rc_size) / 2,
            rc_size,
            rc_size
          ); // 首次绘制内摇杆
        };
        joystick_bg.onload = function() {
          context.drawImage(joystick_bg, 0, 0, rb_size, rb_size);
        };

        this.draw();
      });
    },
    draw() {
      context.clearRect(0, 0, rb_size, rb_size); // 清空画板
      context.drawImage(joystick_bg, 0, 0, rb_size, rb_size); // 画底座
      context.drawImage(
        joystick_cmd,
        centerX - rc_size / 2 + jx,
        centerY - rc_size / 2 + jy,
        rc_size,
        rc_size
      ); // 画摇杆头
      if (!this.defaultOptions.hidden) {
        if (jx > 0 && jy > -5 && jy < 5) {
          // right
          moveForward = false;
          moveLeft = false;
          moveRight = true;
          moveBackward = false;
        } else if (jx < 0 && jy > -5 && jy < 5) {
          // left
          moveForward = false;
          moveLeft = true;
          moveRight = false;
          moveBackward = false;
        } else if (jy > 0 && jx > -5 && jx < 5) {
          // down
          moveForward = false;
          moveLeft = false;
          moveRight = false;
          moveBackward = true;
        } else if (jy < 0 && jx > -5 && jx < 5) {
          // up
          moveForward = true;
          moveLeft = false;
          moveRight = false;
          moveBackward = false;
        } else if (jx > 0 && jy > 0) {
          // right and down
          moveForward = false;
          moveLeft = false;
          moveRight = true;
          moveBackward = true;
        } else if (jx > 0 && jy < 0) {
          // right and up
          moveForward = true;
          moveLeft = false;
          moveRight = true;
          moveBackward = false;
        } else if (jx < 0 && jy > 0) {
          // left and down
          moveForward = false;
          moveLeft = true;
          moveRight = false;
          moveBackward = true;
        } else if (jx < 0 && jy < 0) {
          // left and up
          moveForward = true;
          moveLeft = true;
          moveRight = false;
          moveBackward = false;
        } else {
          moveForward = false;
          moveLeft = false;
          moveRight = false;
          moveBackward = false;
        }
        if (moveForward || moveBackward || moveLeft || moveRight) {
          this.$emit("move", {
            moveForward,
            moveBackward,
            moveLeft,
            moveRight
          });
        }
      }

      requestAnimationFrame(this.draw);
    },
    touch(event) {
      var e = event || window.event;
      switch (e.type) {
        case "touchstart":
          // 判断是否击中摇杆头
          if (
            Math.abs(e.touches[0].clientX - offset_left - jx) >= 0 &&
            Math.abs(e.touches[0].clientX - offset_left - jx) <= 120 &&
            Math.abs(e.touches[0].clientY - offset_height - offset_top - jy) >=
              offset_top &&
            Math.abs(e.touches[0].clientY - offset_height - offset_top - jy) <=
              offset_top + 120
          ) {
            console.log("击中摇杆头");
          }
          break;
        case "touchend": // 手指离开的时候
          // 若手指离开,那就把内摇杆放中间
          jx = 0;
          jy = 0;

          break;
        case "touchmove": // 手指移动的时候：
          // 是否触摸点在摇杆上
          var x, y, r, ans;
          switch (this.defaultOptions.position) {
            case "left-bottom":
              if (
                Math.sqrt(
                  Math.pow(e.touches[0].clientX - offset_left - centerX, 2) +
                    Math.pow(
                      e.touches[0].clientY -
                        offset_height -
                        offset_top -
                        centerY,
                      2
                    )
                ) <=
                rb_size / 2 - rc_size / 2
              ) {
                jx = e.touches[0].clientX - offset_left - centerX;
                jy =
                  e.touches[0].clientY - offset_height - offset_top - centerY;
              } else {
                // 否则计算摇杆最接近的位置
                x = e.touches[0].clientX - offset_left;
                y = e.touches[0].clientY - offset_height - offset_top;
                r = rb_size / 2 - rc_size / 2;

                ans = GetPoint(centerX, centerY, r, centerX, centerY, x, y);
                // 圆与直线有两个交点，计算出离手指最近的交点
                if (
                  Math.sqrt(
                    (ans[0] - x) * (ans[0] - x) + (ans[1] - y) * (ans[1] - y)
                  ) <
                  Math.sqrt(
                    (ans[2] - x) * (ans[2] - x) + (ans[3] - y) * (ans[3] - y)
                  )
                ) {
                  jx = ans[0] - centerX;
                  jy = ans[1] - centerY;
                } else {
                  jx = ans[2] - centerX;
                  jy = ans[3] - centerY;
                }
              }
              break;
            case "right-bottom":
              if (
                Math.sqrt(
                  Math.pow(
                    e.touches[0].clientX -
                      joystick_left -
                      offset_left -
                      centerX,
                    2
                  ) +
                    Math.pow(
                      e.touches[0].clientY -
                        offset_height -
                        offset_top -
                        centerY,
                      2
                    )
                ) <=
                rb_size / 2 - rc_size / 2
              ) {
                jx =
                  e.touches[0].clientX - joystick_left - offset_left - centerX;
                jy =
                  e.touches[0].clientY - offset_height - offset_top - centerY;
              } else {
                // 否则计算摇杆最接近的位置
                x = e.touches[0].clientX - joystick_left - offset_left;
                y = e.touches[0].clientY - offset_height - offset_top;
                r = rb_size / 2 - rc_size / 2;

                ans = GetPoint(centerX, centerY, r, centerX, centerY, x, y);
                // 圆与直线有两个交点，计算出离手指最近的交点
                if (
                  Math.sqrt(
                    (ans[0] - x) * (ans[0] - x) + (ans[1] - y) * (ans[1] - y)
                  ) <
                  Math.sqrt(
                    (ans[2] - x) * (ans[2] - x) + (ans[3] - y) * (ans[3] - y)
                  )
                ) {
                  jx = ans[0] - centerX;
                  jy = ans[1] - centerY;
                } else {
                  jx = ans[2] - centerX;
                  jy = ans[3] - centerY;
                }
              }
              break;

            default:
              break;
          }

          // move();
          e.preventDefault(); // 防止页面滑动，取消掉默认的事件
          break;
      }
      // 计算圆于直线的交点（这一块好难啊）
      function GetPoint(cx, cy, r, stx, sty, edx, edy) {
        var k = (edy - sty) / (edx - stx);
        var b = edy - k * edx;
        var x1, y1, x2, y2;
        var c = cx * cx + (b - cy) * (b - cy) - r * r;
        var a = 1 + k * k;
        var b1 = 2 * cx - 2 * k * (b - cy);

        var tmp = Math.sqrt(b1 * b1 - 4 * a * c);

        x1 = (b1 + tmp) / (2 * a);
        y1 = k * x1 + b;

        x2 = (b1 - tmp) / (2 * a);
        y2 = k * x2 + b;
        return [x1, y1, x2, y2];
      }
    }
  }
};
</script>
<style scoped>
.peach-joystick-container {
  position: absolute;
  display: flex;
  align-items: center;
  justify-content: center;
}
.peach-joystick-left-bottom {
  left: 0;
  bottom: 0;
}
.peach-joystick-right-bottom {
  right: 0;
  bottom: 0;
}
</style>