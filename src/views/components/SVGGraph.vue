<template>
  <div
    id="svgfield"
    :style="{
      width: width + 'px',
      height: height + 'px',
      border: '1px solid grey',
      'border-radius': '5px'
    }"
  >
    <input type="text" v-model="role" />
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "SvgGraph",
  props: ["moduleUri"],
  data() {
    return {
      svgfile: require("../../assets/modcat.svg"),
      width: 1024,
      height: 600,
      gridSize: 100,
      role: "",
      moduleInfo: [],
      modBasicData: [],
      modOutcomes: [],
      modMethods: [],
      modLiterature: [],
      modTeachers: [],
      form: "BasicData"
    };
  },
  created() {
    this.width = window.innerWidth/2.5;
    this.height = window.outerHeight/1.5;
  },
  mounted() {
    d3.xml(this.svgfile).then(xml => {
      var importedNode = document.importNode(xml.documentElement, true);
      d3.select("div#svgfield")
        .classed("svg-container", true)
        .each(function() {
          this.appendChild(importedNode);
        });

      let w = d3.select("svg#svg887").style("width");
      w = parseInt(w.substring( 0, w.length-2));
      let h = d3.select("svg#svg887").style("height");
      h = parseInt(h.substring( 0, h.length-2));
      if (this.width < w) {
        d3.select("svg#svg887").style("width", this.width + "px");
      }
      if (this.height < h) {
        d3.select("svg#svg887").style("height", this.height + "px");
      }

      console.log("svg-width", d3.select("svg#svg887").style("width"))

      var svg = d3.select("svg#svg887");
      var g = svg.append("g");

      g.node().appendChild(svg.select("g").node());

      //this is where my problem happens
      d3.select("svg#svg887").call(
        d3.zoom().on("zoom", function() {
          g.attr("transform", d3.event.transform);
        })
      );

      let relaCenSem = d3.select("#rectSoWiSeXY").node().getBBox().width / 2 - 3;
      console.log("relaCenSem", relaCenSem);
      d3.select("#textSoWiSeXY")
        .attr("text-anchor", "middle")
        .attr("dx", relaCenSem)

      this.styleImportedSVG();
    });
  },
  methods: {
    styleImportedSVG(d) {
      let _this = this;
      d3.select("#nodes")
        .selectAll("g")
        .on("mouseover", null)
        .on("mouseout", null)
        .on("click", null);
      d3.select("#nodes")
        .selectAll("g")
        .classed("selected", false);
      d3.select("#layer1")
        .selectAll("path")
        .style("stroke-width", 0.2);
      d3.select("defs")
        .selectAll("marker")
        .selectAll("path")
        .attr("transform", "scale(0.8) rotate(180) translate(12.5,0)");
      d3.select("#nodes")
        .selectAll("g:not(.faded)")
        .on("mouseover", function() {
          this.style.opacity = 0.5;
          this.style.transition = "0.3s opacity";
        })
        .on("mouseout", function() {
          this.style.opacity = 1;
        })
        .on("click", function() {
          _this.upG();
          //const g = d3.select('#nodes').selectAll("g")
          const g = d3.select("#nodes").selectAll("g");
          g.classed("selected", false);
          d3.select("#layer1")
            .selectAll("path")
            .style("stroke-width", 0.2);
          d3.select("defs")
            .selectAll("marker")
            .selectAll("path")
            .attr("transform", "scale(0.8) rotate(180) translate(12.5,0)");
          //d3.select(this).classed('selected', true)

          let id = this.id;
          console.log("id", id);
          let q = "";

          if (
            /*id == "nodeModulKuerzel" || */ id == "nodeStudiengang" ||
            id == "nodeOrdnung" || id =="nodePerson"
          ) {
            _this.form = "BasicData";
            d3.select("#nodeModulKuerzel").classed("selected", true);
            d3.select("#nodeStudiengang").classed("selected", true);
            d3.select("#nodeOrdnung").classed("selected", true);
            d3.select("#nodePerson").classed("selected", true);
            d3.select("#layer1")
              .selectAll("path.groupA")
              .transition()
              .duration(300)
              .style("stroke-width", "1px"); /*.style('stroke', 'red')*/
            d3.select("defs")
              .selectAll("marker")
              .selectAll("path.groupA")
              .attr(
                "transform",
                "scale(0.3) rotate(180) translate(12.5,0)"
              ); /*.style('stroke', 'red')*/
          } else if (id == "nodeSoWiSeXY") {
            //_this.form = "Teachers";
            d3.select("#nodeModulKuerzel").classed("selected", true);
            d3.select("#nodeSoWiSeXY").classed("selected", true);
            d3.select("#nodePerson").classed("selected", true);
            d3.select("#nodeOrganisation").classed("selected", true);
            d3.select("#nodes")
              .selectAll("g.groupS")
              .classed("faded", false);
            d3.select("#pathLabels")
              .selectAll("g.groupS")
              .classed("faded", false);
            d3.select("#layer1")
              .selectAll("path.groupD")
              .transition()
              .duration(300)
              .style("stroke-width", "1px");
            d3.select("defs")
              .selectAll("marker")
              .selectAll("path.groupD")
              .attr("transform", "scale(0.3) rotate(180) translate(12.5,0)");
            d3.select("#layer1")
              .selectAll("path.groupS")
              .classed("faded", false);
            d3.select("#layer1")
              .selectAll("path.groupS")
              .transition()
              .duration(300)
              .style("stroke-width", "1px");
            d3.select("defs")
              .selectAll("marker")
              .selectAll("path.groupS")
              .attr("transform", "scale(0.3) rotate(180) translate(12.5,0)");
          } else if (id == "nodeDidaktik") {
            _this.form = "Methods";
            d3.select("#nodeModulKuerzel").classed("selected", true);
            d3.select("#nodeDidaktik").classed("selected", true);
            d3.select("#layer1")
              .selectAll("path.groupC")
              .transition()
              .duration(300)
              .style("stroke-width", "1px");
            d3.select("defs")
              .selectAll("marker")
              .selectAll("path.groupC")
              .attr("transform", "scale(0.3) rotate(180) translate(12.5,0)");
            if (_this.modMethods.length == 0) {
              q =
                "PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> " +
                "PREFIX schema: <https://schema.org/> " +
                "PREFIX module: <https://bmake.th-brandenburg.de/module/> " +
                "SELECT * " +
                "WHERE { " +
                "  { " +
                "    SELECT DISTINCT ?code ?label " +
                " WHERE { " +
                "<" +
                _this.moduleUri +
                "> schema:courseCode ?code ; " +
                "   rdfs:label ?label. " +
                " } " +
                "  } UNION { " +
                '     SELECT DISTINCT (GROUP_CONCAT(?interType ;separator="|| ") as ?interTypes) ' +
                " WHERE { " +
                "<" +
                _this.moduleUri +
                "> schema:interactivityType ?interType. " +
                " } " +
                "  } UNION { " +
                "    SELECT DISTINCT (GROUP_CONCAT(CONCAT('[', ?wlname, ',', ?wlvalue,']');separator=\" || \") as ?wlnames) " +
                " WHERE { " +
                "<" +
                _this.moduleUri +
                "> module:addProp_CompWL ?workComp . " +
                "       ?workComp schema:valueReference ?wlList . " +
                "       ?wlList schema:name ?wlname; " +
                "               schema:value ?wlvalue . " +
                " } " +
                "  } " +
                " }";
              _this.queryModuleInfo(q);
            }
          } else if (id == "nodeBeschreibung") {
            _this.form = "Outcomes";
            d3.select("#nodeModulKuerzel").classed("selected", true);
            d3.select("#nodeBeschreibung").classed("selected", true);
            d3.select("#layer1")
              .selectAll("path.groupB")
              .transition()
              .duration(300)
              .style("stroke-width", "1px");
            d3.select("defs")
              .selectAll("marker")
              .selectAll("path.groupB")
              .attr("transform", "scale(0.3) rotate(180) translate(12.5,0)");
            if (_this.modOutcomes.length == 0) {
              q =
                "PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>  " +
                "PREFIX schema: <https://schema.org/>  " +
                "PREFIX module: <https://bmake.th-brandenburg.de/module/>  " +
                "SELECT *  " +
                "WHERE {  " +
                "  {  " +
                "    SELECT DISTINCT ?code ?label  " +
                " WHERE {  " +
                "<" +
                _this.moduleUri +
                "> schema:courseCode ?code ;  " +
                "   rdfs:label ?label.  " +
                " }  " +
                "  } UNION {  " +
                '     SELECT DISTINCT (GROUP_CONCAT(?resList;separator=" || ") as ?resLists)  ' +
                "  WHERE {  " +
                "<" +
                _this.moduleUri +
                "> module:about_LResults ?LResults .  " +
                "       ?LResults schema:itemListElement ?resList .  " +
                "  }  " +
                "  } UNION {  " +
                '      SELECT DISTINCT (GROUP_CONCAT(?conList;separator=" || ") as ?conLists)  ' +
                "  WHERE {  " +
                "<" +
                _this.moduleUri +
                "> module:about_Content ?content.  " +
                "       ?content schema:itemListElement ?conList .  " +
                "  }   " +
                "  } UNION {  " +
                '      SELECT DISTINCT (GROUP_CONCAT(?examList;separator=" || ") as ?examLists)  ' +
                "  WHERE {  " +
                "<" +
                _this.moduleUri +
                "> module:about_Exam ?exam.  " +
                "       ?exam schema:itemListElement ?examList .  " +
                "  }   " +
                "  }  " +
                "}";
              _this.queryModuleInfo(q);
            }
          } else if (id == "nodeLiteratur") {
            //_this.form = "Literature";
            d3.select("#nodeModulKuerzel").classed("selected", true);
            d3.select("#nodeLiteratur").classed("selected", true);
            d3.select("#nodePerson").classed("selected", true);
            d3.select("#nodeOrganisation").classed("selected", true);
            d3.select("#nodes")
              .selectAll("g.groupL")
              .classed("faded", false);
            d3.select("#pathLabels")
              .selectAll("g.groupL")
              .classed("faded", false);
            d3.select("#nodePerson")
              .classed("faded", false);
            d3.select("#layer1")
              .selectAll("path.groupE")
              .transition()
              .duration(300)
              .style("stroke-width", "1px");
            d3.select("defs")
              .selectAll("marker")
              .selectAll("path.groupE")
              .attr("transform", "scale(0.3) rotate(180) translate(12.5,0)");
            d3.select("#layer1")
              .selectAll("path.groupL")
              .classed("faded", false);
            d3.select("#layer1")
              .selectAll("path.groupL")
              .transition()
              .duration(300)
              .style("stroke-width", "1px");
            d3.select("defs")
              .selectAll("marker")
              .selectAll("path.groupL")
              .attr("transform", "scale(0.3) rotate(180) translate(12.5,0)");
          }
        });
    },
    queryModuleInfo(q) {
      let query =
        "PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> " +
        "PREFIX schema: <https://schema.org/> " +
        "SELECT DISTINCT * " +
        "WHERE { " +
        "<" +
        this.moduleUri +
        "> rdfs:label ?label; " +
        "          schema:courseCode ?code; " +
        "          schema:hasCourseInstance ?instance; " +
        "          schema:accountablePerson ?person. " +
        "}";

      console.log(q);
      axios
        .post("http://fbw-sgmwi.th-brandenburg.de:3030/modcat/query", q, {
          headers: { "Content-Type": "application/sparql-query" }
        })
        .then(response => {
          // JSON responses are automatically parsed.
          this.moduleInfo = response.data.results.bindings;
          /*console.log("moduleInfo", this.moduleInfo)*/
        })
        .catch(e => {
          this.errors.push(e);
        });
    },
    updateGraphText() {
      let module = d3.select("#textModulkuerzel").select("tspan");
      let semester = d3.select("#textSoWiSeXY").select("tspan");
      console.log("updateInfo", this.moduleInfo);
      let tModule = "Modul " + this.moduleInfo[0].code.value;
      let tSemester = this.moduleInfo[0].semester.value.substring(39);

      module.text(tModule);
      semester.text(tSemester);

      let relaCenMod = d3.select("#rectModulkuerzel").node().getBBox().width / 2 - 3;
      d3.select("#textModulKuerzel")
        .attr("text-anchor", "middle")
        .attr("dx", relaCenMod);

      let relaCenSem = d3.select("#rectSoWiSeXY").node().getBBox().width / 2 - 3;
      d3.select("#textSoWiSeXY")
        .attr("text-anchor", "middle")
        .attr("dx", relaCenSem);
    },
    upG() {
      if (this.role != "") {
        d3.select("#layer1")
          .selectAll(".faded")
          .classed("faded", false);
        let group = ".group" + this.role;
        d3.select("#layer1")
          .selectAll(group)
          .classed("faded", true);
        d3.select("#layer1")
          .selectAll(".groupS")
          .classed("faded", true);
        d3.select("#layer1")
          .selectAll(".groupL")
          .classed("faded", true);
      } else {
        d3.select("#layer1")
          .selectAll(".faded")
          .classed("faded", false);
      }
    }
  },
  watch: {
    moduleUri: {
      handler(uri) {
        this.queryModuleInfo(
          "PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> " +
            "PREFIX schema: <https://schema.org/> " +
            "PREFIX module: <https://bmake.th-brandenburg.de/module/> " +
            "SELECT * " +
            "WHERE { " +
            "  { " +
            "    SELECT DISTINCT ?code ?label ?curr_name ?curr_des ?modType_name ?grade_name ?grade_des ?sws_name ?ects ?semester ?durationSem ?courseMode ?eduUse ?url ?comment ?pre ?basedOns" +
            "WHERE { " +
            "<" +
            uri +
            "> schema:courseCode ?code ; " +
            "        rdfs:label ?label; " +
            "        module:eduAlignm_Curr ?curr ;  " +
            "        module:eduAlignm_Grade ?grade ; " +
            "        module:eduAlignm_ModuleType ?modType ;  " +
            "        module:eduAlignm_SWS ?sws ; " +
            "        schema:educationalCredentialAwarded ?ects ; " +
            "        schema:hasCourseInstance ?semester ; " +
            "        schema:educationalUse ?eduUse ; " +
            "        schema:coursePrerequisites ?pre ; " +
            "        schema:url ?url ; " +
            "        schema:comment ?comment . " +
            "        OPTIONAL { <" +
            uri +
            ">  schema:coursePrerequisites ?pre } " +
            "        OPTIONAL {  " +
            '           SELECT (GROUP_CONCAT(?basedOn; separator=" || ") as ?basedOns) ' +
            "           WHERE { " +
            "           <" +
            uri +
            ">  schema:isBasedOn ?basedOn . " +
            "           } " +
            "        } " +
            "    ?semester schema:duration ?durationSem; " +
            "             schema:courseMode ?courseMode. " +
            "     ?curr schema:targetName ?curr_name ; " +
            "           schema:targetDescription ?curr_des . " +
            "    ?grade schema:targetName ?grade_name ; " +
            "           schema:targetDescription ?grade_des . " +
            "    ?modType schema:targetName ?modType_name . " +
            "    ?sws schema:targetName ?sws_name .  " +
            "} " +
            "  } UNION { " +
            '    SELECT (GROUP_CONCAT(?language; separator=" || ") as ?languages) ' +
            "    WHERE { " +
            "    module:WIB_AAIT a module:Module; " +
            "    schema:inLanguage ?language . " +
            "} " +
            "  } UNION { " +
            '     SELECT (GROUP_CONCAT(?learnType; separator=" || ") as ?learnTypes) ' +
            "    WHERE { " +
            "    module:WIB_AAIT a module:Module; " +
            "    schema:learningResourceType ?learnType. " +
            "}   " +
            "  } " +
            "}"
        );
        this.modOutcomes = [];
        this.modMethods = [];
        this.modLiterature = [];
        this.modTeachers = [];
        //this.updateGraphText()
      }
    },
    moduleInfo: {
      handler(newData) {
        if (this.form == "BasicData") {
          this.updateGraphText();
          this.modBasicData = this.moduleInfo;
        } else {
          let mod = "mod" + this.form;
          this[mod] = this.moduleInfo;
        }
      }
    },
    modBasicData: {
      handler(newData) {
        if (this.modBasicData.length > 0) {
          this.$emit("modBasicData", newData);
        }
      }
    },
    modOutcomes: {
      handler(v) {
        console.log("modOutcomes", v);
        if (this.modOutcomes.length > 0) {
          this.$emit("modOutcomes", v);
        }
      }
    },
    modMethods: {
      handler(v) {
        console.log("modMethods", v);
        if (this.modMethods.length > 0) {
          this.$emit("modMethods", v);
        }
      }
    },
    modLiterature: {
      handler(v) {
        console.log("modLiterature", v);
        if (this.modLiterature.length > 0) {
          this.$emit("modLiterature", v);
        }
      }
    },
    modTeachers: {
      handler(v) {
        console.log("modTeachers", v);
        if (this.modTeachers.length > 0) {
          this.$emit("modTeachers", v);
        }
      }
    },
    form: {
      handler(v) {
        console.log("form", v);
        this.$emit("formType", v);
      }
    },
    role: {
      handler(v) {
        this.upG();
        this.styleImportedSVG();
      }
    }
  }
};
</script>

<style scoped>
.svg-container {
  display: inline-block;
  position: relative;
  width: 100%;
  vertical-align: top;
  overflow: hidden;
}
</style>
