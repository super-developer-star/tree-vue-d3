<template>
  <div id="diagram">
    <svg></svg>
    <div class="popup" v-show="isShow">
      <div>Test</div>
      <p> {{title}} </p>
      <i class="material-icons" @click="closePopup">close</i>
    </div>
    <div class="hover">Explored</div>
  </div>
</template>

<script>
import * as d3 from 'd3'

export default {
  name: 'topic-tree',
  props: ['data'],
  data: () => ({
    root: null,
    nodes: null,
    center: { x: 0, y: 0 },
    parentRadius: 40,
    childRadius: 20,
    distance: 200,
    distanceRatio: 2,
    between: 50,
    duration: 200,
    counter: 0,
    isShow: false,
    title: ''
  }),
  mounted () {
    this.initialize()
    this.draw()
  },
  methods: {
    closePopup () {
      this.isShow = false
    },
    initialize () {
      this.center.x = document.getElementById('diagram').clientWidth / 2
      this.center.y = document.getElementById('diagram').clientHeight / 2

      this.tree = d3.tree()
      this.root = d3.hierarchy(this.data, (d) => (d.children))
      this.root.children.forEach(this.collapsed)
    },
    collapsed (d) {
      d.clickable = true
      d.isParent = false
      if (d.children) {
        d._children = d.children
        d._children.forEach(this.collapsed)
        if (d.depth > 0) {
          d.children = null
        }
      }
    },
    draw () {
      const that = this

      const svg = d3.select('svg')
      this.panel = d3.select('svg').append('g')
        .attr('class', 'content')
        .attr('transform', `translate(${this.center.x}, ${this.center.y})`)
        .attr('cursor', 'pointer')
      this.linksTag = this.panel.append('g').attr('class', 'links')

      this.zoom = d3.zoom().scaleExtent([0.5, 3]).on('zoom', function () {
        that.panel.attr('transform', d3.event.transform)
      })
      const transform = d3.zoomIdentity.translate(this.center.x, this.center.y).scale(1)
      svg.call(that.zoom.transform, transform)
      svg.call(that.zoom)

      this.update()
    },
    update () {
      const that = this
      const treeData = this.tree(this.root)
      this.nodes = treeData.descendants()
      this.nodes.forEach(function (d) {
        if (d.parent === null) {
          d.sx = d.sy = d.cx = d.cy = 0
        } else {
          d.sx = d.parent.cx
          d.sy = d.parent.cy
        }

        if (d.children && d.children.length) {
          let angle = 360 / d.children.length
          if (d.depth < 1) {
            d.children.forEach(function (ch, i) {
              ch.angle = angle * (i + 1)
              ch.sx = d.sx
              ch.sy = d.sy
            })
          }
        }
      })

      const links = this.linksTag.selectAll('line')
        .data(that.nodes, function (d) {
          return d.id || (d.id = ++that.counter)
        })
      const node = this.panel.selectAll('g.node')
        .data(that.nodes, function (d) {
          return d.id || (d.id = ++that.counter)
        })

      this.drawLine(links)
      this.drawCircle(node)
    },

    // draw the link
    drawLine (links) {
      const that = this
      const nodeEnter = links.enter().append('line')
        .attr('id', d => 'line-' + d.id)
        .attr('x1', d => (d.sx))
        .attr('y1', d => (d.sy))
        .attr('x2', d => (d.sx))
        .attr('y2', d => (d.sy))

      let delayTime = 0

      nodeEnter
        .transition()
        .delay(() => {
          delayTime++
          return delayTime * that.duration
        })
        .duration(3 * that.duration)
        .attr('x1', d => (d.sx))
        .attr('y1', d => (d.sy))
        .attr('x2', function (d) {
          if (d.angle) d.cx = d.sx + that.distance * Math.cos((d.angle / 180) * Math.PI)
          return d.cx
        })
        .attr('y2', function (d) {
          if (d.angle) d.cy = d.sy + that.distance * Math.sin((d.angle / 180) * Math.PI)
          return d.cy
        })

      let removeDelayTime = 0

      links.exit()
        .transition()
        .delay(() => {
          removeDelayTime++
          return removeDelayTime * that.duration
        })
        .duration(3 * that.duration)
        .attr('x1', d => (d.sx))
        .attr('y1', d => (d.sy))
        .attr('x2', function (d) {
          return d.sx
        })
        .attr('y2', function (d) {
          return d.sy
        })
        .remove()
    },

    // draw the circle
    drawCircle (node) {
      const that = this
      const nodeEnter = node.enter()
        .append('g')
        .attr('id', d => 'group-' + d.id)
        .attr('class', 'node')
        .attr('transform', d => 'translate(' + d.sx + ',' + d.sy + ')')
        .style('opacity', 0)

      let delayTime = 0

      nodeEnter.append('circle')
        .attr('class', (d) => (d.parent && d.data.hasChild ? 'hasChild' : ''))
        .attr('r', (d) => ((d.parent) ? that.childRadius : that.parentRadius))
        .on('click', d => {
          if (d.parent && d.data.hasChild && d.clickable) that.nodeClick(d, this)
        })
        .on('mouseover', function (d) {
          if (d.clickable === false) {
            d3.select('#group-' + d.id).selectAll('circle').style('cursor', 'default')
            d3.select('#group-' + d.id).selectAll('text').style('cursor', 'default')
            return false
          } else {
            d3.select('#group-' + d.id).selectAll('circle').style('cursor', 'pointer')
            d3.select('#group-' + d.id).selectAll('text').style('cursor', 'pointer')
          }

          d3.select('.hover')
            .html(() => {
              if (d.data.hasChild && d.isParent) {
                return 'Collapse'
              } else {
                return 'Explore'
              }
            })
            .style('display', 'block')
            .style('left', (d3.event.pageX + 10) + 'px')
            .style('top', (d3.event.pageY + 10) + 'px')
        })
        .on('mouseout', () => {
          d3.select('.hover').style('display', 'none')
        })

      nodeEnter.append('text')
        .html(d => d.data.name)
        .attr('y', (d) => ((d.parent) ? 40 : 60))
        .on('click', d => {
          if (d.clickable) {
            that.isShow = true
            that.title = d.data.name
          }
        })

      nodeEnter
        .transition()
        .delay(() => {
          delayTime++
          return delayTime * that.duration
        })
        .duration(3 * that.duration)
        .attr('transform', d => ('translate(' + d.cx + ',' + d.cy + ')'))
        .style('opacity', 1)

      let removeDelayTime = 0

      node.exit()
        .transition()
        .delay(() => {
          removeDelayTime++
          return removeDelayTime * that.duration
        })
        .duration(3 * that.duration)
        .attr('transform', d => ('translate(' + d.sx + ',' + d.sy + ')'))
        .remove()
    },
    nodeClick (d) {
      if (d.children) {
        d._children = d.children
        d.children = null

        d.isParent = false
        // set expand/collapse statue
        d.expanded = false
        d.clickable = true
        d.parent.clickable = true

        // set show/hide statue
        d.parent.children.map(dt => {
          dt.clickable = true
        })

        this.update()
        this.collapse(d)
      } else {
        d.children = d._children
        d._children = null

        d.isParent = true
        this.expand(d)
      }
    },
    expand (node) {
      const that = this
      node.sx = node.cx
      node.sy = node.cy
      node.cx = node.cx + (this.distance + this.between) * Math.cos((node.angle / 180) * Math.PI)
      node.cy = node.cy + (this.distance + this.between) * Math.sin((node.angle / 180) * Math.PI)

      d3.select('#line-' + node.id)
        .transition()
        .duration(3 * this.duration)
        .attr('x1', d => (d.parent.cx))
        .attr('y1', d => (d.parent.cy))
        .attr('x2', d => {
          if (d.angle) d.cx = d.sx + that.distance * Math.cos((d.angle / 180) * Math.PI)
          return d.cx
        })
        .attr('y2', d => {
          if (d.angle) d.cy = d.sy + that.distance * Math.sin((d.angle / 180) * Math.PI)
          return d.cy
        })
        .on('end', () => {
          that.expandNode(node)
          this.hideOtherNodes(true, node)
        })

      d3.select('#group-' + node.id)
        .transition()
        .duration(3 * this.duration)
        .attr('transform', 'translate(' + node.cx + ',' + node.cy + ')')
      d3.select('#group-' + node.id).selectAll('circle')
        .transition()
        .duration(3 * this.duration)
        .attr('r', that.parentRadius)
        .attr('class', '')
      d3.select('#group-' + node.id).selectAll('text')
        .transition()
        .duration(3 * this.duration)
        .attr('y', 60)
    },
    expandNode (d) {
      const radio = 360 / (d.children.length + 1)

      // set show/hide statue
      d.parent.children.map(dt => {
        dt.clickable = false
      })

      // set expand/collapse statue
      d.expanded = true
      d.clickable = true
      d.parent.clickable = false

      // set children's angel
      d.children.map((v, i) => {
        v.angle = d.angle + 180 + (i + 1) * radio
      })

      this.update()
    },
    collapse (node) {
      const that = this
      node.sx = node.parent.cx
      node.sy = node.parent.cy
      node.cx = node.cx + (this.distance + this.between) * Math.cos((node.angle / 180) * Math.PI)
      node.cy = node.cy + (this.distance + this.between) * Math.sin((node.angle / 180) * Math.PI)
      this.hideOtherNodes(false, node)

      d3.select('#line-' + node.id)
        .transition()
        .delay((node._children.length + 3) * that.duration)
        .duration(3 * this.duration)
        .attr('x1', d => (d.parent.cx))
        .attr('y1', d => (d.parent.cy))
        .attr('x2', function (d) {
          if (d.angle) d.cx = d.sx + that.distance * Math.cos((d.angle / 180) * Math.PI)
          return d.cx
        })
        .attr('y2', function (d) {
          if (d.angle) d.cy = d.sy + that.distance * Math.sin((d.angle / 180) * Math.PI)
          return d.cy
        })

      d3.select('#group-' + node.id)
        .transition()
        .delay((node._children.length + 3) * that.duration)
        .duration(3 * this.duration)
        .attr('transform', function (d) {
          return 'translate(' + node.cx + ',' + node.cy + ')'
        })
      d3.select('#group-' + node.id).selectAll('circle')
        .transition()
        .delay((node._children.length + 3) * that.duration)
        .duration(3 * this.duration)
        .attr('r', that.childRadius)
        .attr('class', (d) => (d.data.hasChild ? 'hasChild' : ''))
      d3.select('#group-' + node.id).selectAll('text')
        .transition()
        .delay((node._children.length + 3) * that.duration)
        .duration(3 * this.duration)
        .attr('y', 40)
    },
    hideOtherNodes (isHide, node) {
      const that = this

      if (isHide) {
        // other Nodes hide
        node.parent.children.forEach(function (d) {
          d3.select('#line-' + d.id)
            .transition()
            .duration(3 * that.duration)
            .style('opacity', 0.2)
          d3.select('#group-' + d.id).selectAll('text')
            .transition()
            .duration(3 * that.duration)
            .style('opacity', 0.2)
          d3.select('#group-' + d.id)
            .transition()
            .duration(3 * that.duration)
            .style('opacity', 0.5)
        })

        d3.select('#line-' + node.id)
          .transition()
          .duration(3 * that.duration)
          .style('opacity', 1)
        d3.select('#group-' + node.id).selectAll('text')
          .transition()
          .duration(3 * that.duration)
          .style('opacity', 1)
        d3.select('#group-' + node.id)
          .transition()
          .duration(3 * that.duration)
          .style('opacity', 1)
      } else {
        // other Nodes hide
        node.parent.children.forEach(function (d) {
          d3.select('#line-' + d.id)
            .transition()
            .duration(3 * that.duration)
            .style('opacity', 1)
          d3.select('#group-' + d.id).selectAll('text')
            .transition()
            .duration(3 * that.duration)
            .style('opacity', 1)
          d3.select('#group-' + d.id)
            .transition()
            .duration(3 * that.duration)
            .style('opacity', 1)
        })
      }
    }
  }
}
</script>

<style lang="scss" scoped>
#diagram {
  width: 100%;
  height: 100%;
  position: fixed;
}

svg {
  position: relative;
  width: 100%;
  height: 100%;
  background: #f9f9f5;
}

.buttons {
  position: absolute;
  top: 0;
  right: 0;

  i {
    background: white;
    box-shadow: 0 0 10px gray;
    cursor: pointer;
    margin: 10px;
  }
}

.popup {
  width: 500px;
  height: 100px;
  background: #ffffff;
  position: fixed;
  bottom: 10px;
  left: 10px;
  box-shadow: 0 0 10px #afafaf;
  border: 1px solid #ccc;
  border-radius: 4px;

  i {
    position: absolute;
    right: 0;
    top: 0;
    cursor: pointer;
    color: black;
    padding: 4px;
  }
}

.hover {
  position: absolute;
  background: #ffffff;
  box-shadow: 0 0 10px #afafaf;
  border: 1px solid #ccc;
  border-radius: 4px;
  top: 0;
  left: 0;
  padding: 5px 10px;
  font-size: 14px;
  display: none;
}

</style>

<style lang="scss">
circle {
  fill: #8ee000;
  cursor: default;
}

.hasChild {
  stroke: red;
  cursor: pointer;
}

line {
  stroke: #e0ddd5;
  stroke-width: 2px;
}

text {
  text-anchor: middle;
  opacity: 1;
}
</style>
