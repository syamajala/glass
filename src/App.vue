<template>
  <div id="app">
    <div id="plot"></div>
    X:
    <el-select v-model="xCol" placeholder="Select" @change="plot" clearable>
      <el-option v-for="(col, idx) in cols" :key="idx" :label="col" :value="col"></el-option>
    </el-select>
    Y:
    <el-select v-model="yCol" placeholder="Select" @change="plot" clearable>
      <el-option v-for="(col, idx) in cols" :key="idx" :label="col" :value="col"></el-option>
    </el-select>
    <el-table
      ref="glassTable"
      :data="glass"
      border
      highlight-current-row
      @row-click="handleRowClick"
      height="500"
      style="width: 100%">
      <el-table-column v-for="(col, idx) in cols" :key="idx" :prop="col" :label="col" sortable>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import * as d3 from 'd3-request'
const Plotly = require('plotly.js/dist/plotly.min.js')

export default {

data () {
    return {
        glass: [],
        cols: [],
        xCol: "",
        yCol: "",
        traces: new Map(),
        currentRow: null
    }
},

mounted() {
    Plotly.newPlot('plot', [], {});

    d3.csv("/glass.csv", (data) => {
        this.glass = data;
        this.cols = Object.keys(data[0]);

        let colors = ['rgb(102,194,165)',
                      'rgb(252,141,98)',
                      'rgb(141,160,203)',
                      'rgb(231,138,195)',
                      'rgb(166,216,84)',
                      'rgb(255,217,47)',
                      'rgb(229,196,148)',
                      'rgb(179,179,179)']

        var types = new Set();

        this.glass.map((row) => {
            types.add(row['type'])
        });

        types = [...types].sort();

        types.map((typ) => {
            this.traces.set(typ, {
                x: [],
                y: [],
                mode: 'markers',
                type: 'scatter',
                text: [],
                name: 'Type ' + typ,
                marker: { color: colors[typ-1] }
            })
        });
    });
},

methods: {
    plot() {
        var x_vals = [];
        var y_vals = [];
        var text = [];

        this.traces.forEach((val) => {
            val.x = [];
            val.y = [];
            val.text = [];
        })

        this.glass.map((row) => {
            let trace = this.traces.get(row.type);

            trace.x.push(row[this.xCol]);
            trace.y.push(row[this.yCol]);
            let s = "ID: " + row['id'] + "<br>" + "Type: " + row['type']
            trace.text.push(s);
        });

        var layout = {
            title: this.xCol + ' vs ' + this.yCol,
            xaxis: { title: this.xCol },
            yaxis: { title: this.yCol },
            hovermode: 'closest'
        };

        Plotly.newPlot('plot', Array.from(this.traces.values()), layout);
    },

    handleRowClick(row) {
        var plotDiv = document.getElementById('plot');

        if (row == this.currentRow || (row != this.currentRow && this.currentRow != null))
        {
            this.currentRow = null;
            this.$refs.glassTable.setCurrentRow();
            Plotly.deleteTraces(plotDiv, -1)
        }
        else
        {
            this.currentRow = row;
            this.$refs.glassTable.setCurrentRow(row);
            Plotly.addTraces(plotDiv, [{
                x: [row[this.xCol]],
                y: [row[this.yCol]],
                mode: 'markers',
                type: 'scatter',
                name: 'ID ' + row['id'],
                text: ["ID: " + row['id'] + "<br>" + "Type: " + row['type']],
                marker: { color: 'black', size: 10 }
            }])
        }
    }
}
}
</script>

<style>
body {
  font-family: Helvetica, sans-serif;
}
</style>
