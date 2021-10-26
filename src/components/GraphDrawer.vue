<template>
  <v-container class="pa-6">
    <v-row class="mb-4">
      <v-btn
        v-if="!add_link"
        outlined
        class="mr-2"
        :color="add_vert ? 'error' : 'primary'"
        @click="add_vert = !add_vert"
      >
        {{ add_vert ? 'Отмена' : 'Создать вершину' }}
      </v-btn>
      <v-btn
        v-if="!add_vert"
        outlined
        :color="add_link ? 'error' : 'primary'"
        @click="link_parental = null; add_link = !add_link"
      >
        {{ add_link ? 'Отмена' : 'Создать связь' }}
      </v-btn>
    </v-row>
    <v-row>
      <canvas
        ref="graph_canva"
        width="600px"
        height="300px"
        @click="clickHandler"
      > Извините, ваш браузер нет поддерживает&lt;canvas&gt; элемент. </canvas>
    </v-row>
    <v-row v-if="verticies.length != 0">
      <v-simple-table>
        <template v-slot:default>
          <thead>
            <tr>
              <th>Узел</th>
              <th>Наименование работы</th>
              <th>Код</th>
              <th>Длительность (часы)</th>
              <th>Стоимость</th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="vert in verticies"
              :key="`vert-${vert.x}-${vert.y}`"
            >
              <td class="pa-4">
                <div class="text-h6" :style="{color: $vuetify.theme.themes.light.primary}">
                  {{ vert.init ? 'S' : '' }}{{ vert.childrens.length == 0 ? 'F' : '' }}
                </div>
              </td>
              <td class="pa-4">
                <v-text-field
                  outlined
                  label="Наименование работы"
                  v-model="vert.name"
                  hide-details
                ></v-text-field>
              </td>
              <td class="pa-4">
                <v-text-field
                  outlined
                  label="Код"
                  @input="redraw()"
                  v-model="vert.code"
                  hide-details
                ></v-text-field>
              </td>
              <td class="pa-4">
                <v-text-field
                  outlined
                  label="Длительность"
                  suffix="Часов"
                  v-model="vert.duration"
                  hide-details
                ></v-text-field>
              </td>
              <td class="pa-4">
                <v-text-field
                  outlined
                  label="Стоимость"
                  suffix="Руб."
                  v-model="vert.price"
                  hide-details
                ></v-text-field>
              </td>
              <td class="pa-4">
                <v-btn
                  icon
                  color="error"
                  @click="removeVert(vert)"
                >
                  <v-icon> mdi-delete-outline </v-icon>
                </v-btn>
              </td>
            </tr>
          </tbody>
        </template>
      </v-simple-table>
      <v-btn
        outlined
        color="primary"
        class="mt-4"
        @click="getPaths()"
      > Расчитать </v-btn>
    </v-row>
    <v-row v-if="result.length > 0" class="pt-6">
      <v-expansion-panels>
        <v-expansion-panel>
          <v-expansion-panel-header>
            Таблица путей (всего путей: {{ result.length }})
          </v-expansion-panel-header>
          <v-expansion-panel-content>
            <v-simple-table style="width: 100%;">
              <template v-slot:default>
                <thead>
                  <th>Путь </th>
                  <th>Итого часов </th>
                  <th>Итого Руб </th>
                </thead>
                <tbody>
                  <tr
                    v-for="path in result"
                    :key="`result-${path.length}`"
                  >
                    <td>
                      {{ getPathCodes(path) }}
                    </td>
                    <td>
                      {{ getPathEstimates(path) }}
                    </td>
                    <td>
                      {{ getPathCost(path) }}
                    </td>
                  </tr>
                </tbody>
              </template>
            </v-simple-table>
          </v-expansion-panel-content>
        </v-expansion-panel>
        <v-expansion-panel>
          <v-expansion-panel-header>
            График стоимости к длительности
          </v-expansion-panel-header>
          <v-expansion-panel-content>
            <chart
              :data="result.map(path => [getPathEstimates(path), getPathCost(path)])"
            ></chart>
          </v-expansion-panel-content>
        </v-expansion-panel>
      </v-expansion-panels>
    </v-row>
  </v-container>
</template>

<script>
import Chart from './Chart.vue'
export default {
  name: "GraphDrawer",
  components: {
    Chart,
  },
  data: () => ({
    ctx: null,
    add_vert: false,
    add_link: false,
    link_parental: null,
    verticies: [],
    result: [],
  }),
  mounted() {
    let canva = this.$refs.graph_canva;
    this.ctx = canva.getContext('2d');
    this.redraw()
  },
  updated() {
    this.redraw()
  },
  methods: {
    get_chart_opts() {
      return {
        width: 800,
        height: 500,
        chart: {
          title: 'Cтоимость к длительности',
        },
        hAxis: {title: 'Временные затраты, часов'},
        vAxis: {title: 'Финансовые затраты, руб.'}
    }},
    getPathCodes(path) {
      let codes = path.map(item => item.code)
      return codes.join(', ')
    },
    getPathEstimates(path) {
      let est = 0;
      path.forEach(item => est += Number(item.duration))
      return est
    },
    getPathCost(path) {
      let cost = 0;
      path.forEach(item => cost += Number(item.price))
      return cost
    },
    paths(verts, path) {
      if (verts.length == 0) { this.result.push(path.slice(0)); return }
      else {
        for (let i = 0; i < verts.length; i++) {
          path.push(verts[i]);
          var newPath = path
          this.paths(verts[i].childrens, newPath)
          path.pop()
        }
      }
    },
    getPaths() {
      this.result = []
      let start = this.verticies.find(vert => vert.init)
      var path = [start]
      this.paths(start.childrens, path);
    },
    removeVert(removable_vert) {
      this.verticies.splice(this.verticies.indexOf(removable_vert), 1)
      this.verticies.forEach(vert => {
        let child = vert.childrens.find(child => child.x == removable_vert.x && child.y == removable_vert.y)
        if (child === undefined) return
        vert.childrens.splice(vert.childrens.indexOf(child), 1)
      })
      this.redraw()
    },
    redraw() {
      this.ctx.clearRect(0, 0, this.$refs.graph_canva.width, this.$refs.graph_canva.height);
      this.ctx.fillStyle = "rgba(25, 118, 210, 0.24)";
      this.ctx.strokeStyle = "rgba(25, 118, 210)";
      this.ctx.font = '16px Roboto';
      this.verticies.forEach(vert => {
        this.drawVert(vert)
        vert.childrens.forEach(child => {
          this.drawLink(vert, child);
        })
      });
    },
    clickHandler(event) {
      let x = event.offsetX;
      let y = event.offsetY;
      if (this.add_vert) {
        this.verticies.push({
          init: this.verticies.length == 0,
          x: x,
          y: y,
          childrens: [],
          name: '',
          code: '',
          duration: 1,
          price: 100
        })
        this.redraw()
        this.add_vert = false;
      }
      if (this.add_link) {
        if (this.link_parental === null) {
          this.link_parental = this.searchVert(x,y);
          if (this.link_parental === null || this.link_parental === undefined) return;
        } else {
          let child = this.searchVert(x,y);
          if (child === undefined) return;
          if (child == this.link_parental || this.link_parental === null || this.link_parental === undefined) return;
          this.link_parental.childrens.push(child);
          this.redraw();
          this.link_parental = null;
          this.add_link = false;
        }
      }
    },
    searchVert(x,y) {
      return this.verticies.find(vert =>
        (Math.pow((x - vert.x), 2) + Math.pow((y - vert.y), 2)) <= 625
      )
    },
    drawVert(vert) {
      let canva = this.$refs.graph_canva;
      if (canva.height - vert.y <= 100) { canva.height += 100; this.redraw(); }

      this.ctx.beginPath();
      this.ctx.arc(vert.x, vert.y, 25, 0, 360);
      this.ctx.stroke();
      this.ctx.fill();
      this.ctx.fillStyle = "rgb(44, 44, 44)";
      let dx = 8 * (vert.code.length - 1) <= 24 ? 8 * (vert.code.length - 1) : 24 ;
      this.ctx.fillText(vert.code, vert.x - dx, vert.y + 4, 50);
      this.ctx.fillStyle = "rgba(25, 118, 210, 0.24)";
      this.ctx.closePath();
    },
    drawLink(parent, child) {
      let dx = parent.x - child.x;
      let dy = parent.y - child.y;
      let angle = Math.atan2(dy, dx);

      let x1 = parent.x - 25 * Math.cos(angle - Math.PI / 48)
      let y1 = parent.y - 25 * Math.sin(angle - Math.PI / 48)
      let x2 = child.x + 25 * Math.cos(angle + Math.PI / 48)
      let y2 = child.y + 25 * Math.sin(angle + Math.PI / 48)

      this.ctx.moveTo(x1, y1);
      this.ctx.lineTo(x2, y2);

      this.ctx.lineTo(x2 + 10 * Math.cos(angle - Math.PI / 6), y2 + 10 * Math.sin(angle - Math.PI / 6));
      this.ctx.moveTo(x2, y2);
      this.ctx.lineTo(x2 + 10 * Math.cos(angle + Math.PI / 6), y2 + 10 * Math.sin(angle + Math.PI / 6));

      this.ctx.stroke();
      this.ctx.closePath();

    }
  }
}
</script>

<style scoped>
canvas {
  border: solid 1px rgba(25, 118, 210, 0.24);
  border-radius: 4px;
}
</style>