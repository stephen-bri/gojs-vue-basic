<script setup>
import { onMounted, ref } from 'vue'
import * as go from 'gojs'

const props = defineProps({ nodeDataArray: Array, linkDataArray: Array })
const emitter = defineEmits([
  "ExternalObjectsDropped",
  "SelectionMoved"
])

const diagram = ref(null)

onMounted(function() {
  const $ = go.GraphObject.make;

  const myDiagram =
    new go.Diagram(diagram.value,
      {
        "undoManager.isEnabled": true
      });
  
  myDiagram.addDiagramListener("ExternalObjectsDropped", e => emitter("ExternalObjectsDropped", e))
  myDiagram.addDiagramListener("SelectionMoved", e => emitter("SelectionMoved", e))

  myDiagram.groupTemplateMap.add("table", 
				$(go.Group, "Auto", 									// New group sized automatically 
					{
						ungroupable: true,								// Can not add this group into a parent group 
						layout: $(go.GridLayout,{   					// New sub-grid layout 
							wrappingColumn: 	1,						// One column without wrapping 
							alignment: 			go.GridLayout.Position,	// Place by position method 
							cellSize: 			new go.Size(1, 1),		// Grid sizing 
							//spacing: 			new go.Size(0, 0),      // Padding 
						}),
						computesBoundsAfterDrag: true,					// Don't recompute bounds while dragging is occurring 
						layer: 					"background",			// The group is just in the background.  Doesn't interact with anything 
						avoidable: 				false,					// No reason to go around the group (links go through it)
						deletable: 				false,					// Can't delete it 
						copyable: 				false,					// Can't copy the group ... not logical spock 
						movable:				false,
					},	
					// Create the nested objects within the group 
					$(go.Panel, "Vertical",								
						{
							//background: "green", 						//TODO: gid rid of me when done tweaking layout & margins
							alignment: go.Spot.Top,
							stretch: go.GraphObject.Vertical,
							//width:	"100%",
							//stretch: go.GraphObject.Horizontal 			// Doesn't seem to do anything
						},
						$(go.Placeholder,
							{ 
								padding: 0,
								stretch: go.GraphObject.Horizontal      // Doesn't seem to do anything
	
							}
						),
					),
				)
			);


  myDiagram.nodeTemplate =
    $(go.Node, "Auto",
      $(go.Shape,
        { fill: "white" },
        new go.Binding("fill", "color")),
      $(go.TextBlock,
        { margin: 8 },
        new go.Binding("text"))
    );

  myDiagram.linkTemplate =
    $(go.Link,
      {
        relinkableFrom: true, relinkableTo: true,
        reshapable: true, resegmentable: true
      },
      $(go.Shape),
      $(go.Shape, { toArrow: "OpenTriangle" })
    );

  const nda = props.nodeDataArray;
  const lda = props.linkDataArray;
  myDiagram.model = new go.GraphLinksModel(nda, lda);
})
</script>

<template>
  <div ref="diagram" class="goDiagram"></div>
</template>

<style scoped>
.goDiagram {
  width: 400px;
  height: 400px;
  border: solid black 1px;
}
</style>
