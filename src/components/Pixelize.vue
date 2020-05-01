<template>
  <div class="pixelize">
    <h1>Pixelize.vue</h1>
    <div>
      <label>
        <input type="file" v-on:change="upload" accept="image/*" />
      </label>
    </div>
    <p>
      <label>
        色の数
        <input type="number" v-model="colors" />
      </label>
    </p>
    <p>
      <label>
        変更後の幅
        <input type="number" v-model="width_after" />
      </label>
    </p>

    <p>{{ colors }}</p>
    <p>{{ width_after }}</p>
    <p>オリジナル画像の幅：{{ width_before }}</p>
    <p>オリジナル画像の高さ：{{ height_before }}</p>
    <pre>
      {{ $data }}
    </pre>
    <div>
      <h2>変換前プレビュー</h2>
      <canvas id="preview_before"></canvas>
    </div>
  </div>
</template>

<script>
export default {
  name: "Pixelize",
  data() {
    return {
      width_before: 0,
      height_before: 0,
      width_after: 300,
      height_after: 300,
      colors: 0
    };
  },
  methods: {
    upload: function(event) {
      let img = null;
      let file = event.target.files;
      let reader = new FileReader();
      reader.readAsDataURL(file[0]);

      // ファイルを読み込んだときの処理
      reader.onload = function() {
        img = new Image();
        // 画像が読み込まれたときの処理（canvasに描画）
        img.onload = function() {
          let canvas = document.getElementById("preview_before");
          if (canvas.getContext) {
            // canvas 2d contextが使える前提
            let context = canvas.getContext("2d");
            this.width_before = img.width;
            this.height_before = img.height;
            let scale = this.width_after / this.width_before;
            canvas.width = this.width_before * scale;
            canvas.height = this.height_before * scale;
            context.scale(scale, scale);
            context.drawImage(img, 0, 0);
            // ここから変換処理
            let srcData = context.getImageData(
              0,
              0,
              canvas.width,
              canvas.height
            );
            let dstData = context.createImageData(canvas.width, canvas.height);
            let src = srcData.data;
            let dst = dstData.data;
            grayFilter(src, dst, canvas.width, canvas.height);
            context.putImageData(dstData, canvas.width / 2, 0);
          }
        }.bind(this);
        img.src = reader.result;
      }.bind(this);
    }
  }
};

//グレースケール変換関数（仮）
let grayFilter = function(src, dst, width, height) {
  for (var i = 0; i < height; i++) {
    for (var j = 0; j < width; j++) {
      var idx = (j + i * width) * 4;
      var gray = (src[idx] + src[idx + 1] + src[idx + 2]) / 3;
      dst[idx] = gray;
      dst[idx + 1] = gray;
      dst[idx + 2] = gray;
      dst[idx + 3] = src[idx + 3];
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
