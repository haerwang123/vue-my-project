<template>
  <div class="register">
    <el-container>
      <div id="treeId" class="tree-svg" />
    </el-container>
  </div>
</template>

<script>
import d3 from 'd3'
var diagonal = d3.svg.diagonal()
  .projection(function(d) { return [d.x, d.y] })
export default {
  name: 'Register',
  components: {

  },
  data() {
    return {
      root: {
        'name': 'flare',
        'size': '1',
        'image': 'https://wx3.sinaimg.cn/mw690/6405dd36gy1gc2133877dj21ok2cskjm.jpg',
        'children': [
          {
            'name': 'AAA',
            'size': '11',
            'image': 'https://wx4.sinaimg.cn/mw690/c46b3efaly1gc24s5qnskj21400u0qv5.jpg'
          },
          {
            'name': 'BBB',
            'size': '12',
            'image': 'https://wx1.sinaimg.cn/mw690/77de9208ly1gc1qwcage3j20hs0m8mxm.jpg'
          }
        ]
      },
      tree: null,
      zm: null,
      height: 650,
      count: 0,
      duration: 600
    }
  },
  computed: {

  },
  created() {
    var that = this
    // eslint-disable-next-line eqeqeq
    var tree = d3.layout.tree().nodeSize([200, 400]).separation(function(a, b) { return (a.parent == b.parent ? 1.1 : 1.1) })
    that.tree = tree
  },
  mounted() {
    var that = this
    var svg = d3.select('#treeId').append('svg').attr('width', 1200).attr('height', 600)
      .call(that.zm = d3.behavior.zoom().scaleExtent([1, 4]))
      .call(that.zm = d3.behavior.zoom().scaleExtent([1, 3]).on('zoom', d => { svg.attr('transform', 'translate(' + d3.event.translate + ')') }))
      .append('g')
      .attr('transform', 'translate(' + 480 + ',' + 100 + ')')

    that.zm.translate([100, 200])
    that.svg = svg
    that.root.x0 = 110
    that.root.y0 = that.height / 2

    function collapse(d) {
      if (d.children) {
        d._children = d.children
        d._children.forEach(collapse)
        d.children = null
      }
    }
    // Initialize the display to show a few nodes.
    that.root.children.forEach(collapse)
    that.update(that.root)
  },
  methods: {
    update(source) {
      var that = this
      // Compute the new tree layout.
      var nodes = that.tree.nodes(that.root).reverse()
      var links = that.tree.links(nodes)

      // Normalize for fixed-depth.
      nodes.forEach(function(d) {
        d.y = d.depth * 300
      })
      // Update the nodes…
      var node = that.svg.selectAll('g.node')
        .data(nodes, function(d) { return d.id || (d.id = ++that.count) })

      // Enter any new nodes at the parent's previous position.
      var nodeEnter = node.enter().append('g')
        .attr('class', 'node')
        .attr('transform', function(d) {
          return 'translate(' + source.x0 + ',' + source.y0 + ')'
        })
        .on('click', d => that.click(d))

      nodeEnter.append('image')
        .attr('xlink:href', d => {
          return d.image
        })
        .attr('x', d => {
          return d.children || d._children ? -100 : -100
        })
        .attr('y', -100)

      // nodeEnter.append('text')
      //   .attr('x', function(d) { return d.children || d._children ? -200 : -20 })
      //   .attr('y', '15')
      //   .attr('font-size', '18px')
      //   .text(function(d) { return d.name })
      //   .style('fill-opacity', 1e-6)

      // Transition nodes to their new position.
      var nodeUpdate = node.transition()
        .duration(that.duration)
        .attr('transform', function(d) { return 'translate(' + d.x + ',' + d.y + ')' })

      nodeUpdate.selectAll('image')
        .attr('width', 200)
        .attr('height', 350)

      nodeUpdate.select('text')
        .style('fill-opacity', 1)

      // Transition exiting nodes to the parent's new position.
      var nodeExit = node.exit().transition()
        .duration(that.duration)
        .attr('transform', function(d) { return 'translate(' + source.x + ',' + source.y + ')' })
        .remove()

      nodeExit.select('image')
        .attr('width', 0)
        .attr('height', 0)

      nodeExit.select('text')
        .style('fill-opacity', 1e-6)

      // Update the links…
      var link = that.svg.selectAll('path.link')
        .data(links, function(d) {
          return d.target.id
        })

      // link.distance([60])
      // Enter any new links at the parent's previous position.
      link.enter().insert('svg:path', 'g')
        .attr('class', 'link')
        .attr('fill', 'none')
        .attr('stroke', 'blue')
        .attr('stroke-width', '1')
        .attr('d', function(d) {
          var o = { x: source.x0, y: source.y0 }
          return diagonal({ source: o, target: o })
        })

      // Transition links to their new position.
      link.transition()
        .duration(that.duration)
        .attr('d', diagonal)

      // Transition exiting nodes to the parent's new position.
      link.exit().transition()
        .duration(that.duration)
        .attr('d', function(d) {
          var o = { x: source.x, y: source.y }
          return diagonal({ source: o, target: o })
        })
        .remove()

      // Stash the old positions for transition.
      nodes.forEach(function(d) {
        d.x0 = d.x
        d.y0 = d.y
      })
    },
    click(d) {
      var that = this
      // #重点关注这个函数的不同之处。尤其是else部分

      if (d.children) {
        d._children = d.children
        d.children = null
      } else if (d._children) {
        d.children = d._children
        d._children = null
      } else {
        var mnodes = that.getNode(d.size)
        d.children = mnodes
      }
      that.update(d)
    },
    getNode(id) {
      // #自定义的一个新的以同步方式从后台取数据的ajax函数
      var mynodes = mynodes = [{
        'name': 'DDD',
        'image': 'https://wx4.sinaimg.cn/mw690/68418ffbly1gc1lt839zbj20u014077s.jpg',
        'size': '131'
      }, {
        'name': 'EEE',
        'image': 'https://wx1.sinaimg.cn/mw690/90ca507fly1gc1vdlhjfej21400u0wla.jpg',
        'size': '132'
      }, {
        'name': 'FFF',
        'image': 'https://wx2.sinaimg.cn/mw690/61fd0433ly1gc1xcajkw7j20jg0kudq6.jpg',
        'size': '133'
      }]
      return mynodes
    }
  }
}
</script>

<style scoped>
.register {
  padding-top: 50px;
  width: 100%;
}
.tree-svg{
  margin: 0 auto;
  border: 1px solid #f00;
}
.link{
  fill: none;
  stroke: black;
  stroke-width: 1.5px;
}

</style>
