<template>
  <div class="blockly-module">
    <div class="d-flex w-100 h-100 outer-wrap">
      <div class="d-flex flex-fill flex-column main-panel">
        <div
          class="d-flex flex-fill flex-column"
          style="background-color: white"
        >
          <div class="blockly" ref="blocklyDiv"></div>
        </div>
        <div style="height: 200px">
          <div
            class="bg-secondary text-light px-3 py-2 scroll-box"
            ref="logsBox"
          >
            <br />
            <div v-html="result" />
            <div v-html="logs" />
          </div>
        </div>
      </div>

      <div class="side-panel">
        <div>
          <div class="display-panel" v-show="!isExpanded && isRunning">
            <p class="display-image">
              <b-img ref="displayImage" src=""> </b-img>
            </p>
            <div class="control">
              <b-form-checkbox
                class="check"
                v-model="nDisplay"
                name="check-button"
                switch
              >
                Detector
              </b-form-checkbox>
              <img
                src="~/assets/images/UI/svg/expad-arrows.svg"
                height="20"
                v-b-modal.expanded-camera
              />
            </div>
          </div>
        </div>
        <div class="buttom">
          <button
            v-if="!isRunning"
            :disabled="isProjectLoaded"
            pill
            v-on:click="run"
            class="btn-run op-btn"
          >
            <span class="ico"
              ><img src="~/assets/images/UI/svg/Group 80.svg" alt="" srcset=""
            /></span>
          </button>

          <!-- <button
            v-if="!isRunning"
            :disabled="isProjectLoaded"
            pill
            v-on:click="$emit('save-workspace')"
            class="btn-run"
            style="margin-left:10px;">
            <span class="ico">
              <img src="../assets/UI/svg/save.svg" alt="" srcset=""/>
            </span>
          </button> -->

          <button
            v-if="isRunning"
            :disabled="!isProjectLoaded"
            pill
            v-on:click="stop"
            class="btn-stop"
          >
            <span class="ico"
              ><img src="~/assets/images/UI/svg/Group 82.svg" alt="" srcset=""
            /></span>
          </button>
        </div>
      </div>
      <b-modal
        id="expanded-camera"
        size="xl"
        title="Live view"
        modal-class="my-modal-class my-modal-class-no-pad"
        :centered="true"
        :hide-footer="true"
        :hide-header-close="true"
        @shown="setExpanded(true)"
        @hide="setExpanded(false)"
        :no-close-on-backdrop="true"
        :no-close-on-esc="true"
      >
        <div class="display-image">
          <b-img class="realtime-image" ref="displayImageFull" src=""> </b-img>
          <div class="control">
            <b-form-checkbox
              class="check"
              v-model="nDisplay"
              name="check-button"
              switch
            >
              Detector
            </b-form-checkbox>
            <img
              src="~/assets/images/UI/svg/minimize-arrows.svg"
              height="20"
              @click="$bvModal.hide('expanded-camera')"
            />
          </div>
        </div>
      </b-modal>
    </div>
  </div>
</template>

<script>
/**
 * @license
 *
 * Copyright 2019 Google LLC
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *   http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

/**
 * @fileoverview Blockly Vue Component.
 * @author samelh@google.com (Sam El-Husseini)
 */

import Blockly from "blockly";
import blocklyPython from "blockly/python";
import { mapState, mapActions, mapMutations , mapGetters } from "vuex";
export default {
  name: "BlocklyComponent",
  props: {
    /*isRunHiden: Boolean,*/
    isProjectLoaded: Boolean,
    isRunning: Boolean,
  },
  data() {
    return {
      blockly_woakspace: null,
      blockly_xml: "",
      nDisplay: false,
      result: "",
      logs: "",
      s_result: "",
      isExpanded: false,
    };
  },
  methods: {
    async run(){

    },
    async stop(){

    },
    setExpanded: function (value) {
      if (value) {
        this.$refs.displayImageFull.src = this.$refs.displayImage.src;
      } else {
        this.$refs.displayImage.src = this.$refs.displayImageFull.src;
      }
      this.isExpanded = value;
    },
  },
  computed: {
    ...mapState(["currentDevice","serverUrl","streamUrl"]),
    updateXML: function () {
      console.log("Update XML to workspace");
      this.blockly_woakspace.clear();
      let textToDom = Blockly.Xml.textToDom(this.blockly_xml);
      Blockly.Xml.domToWorkspace(this.blockly_woakspace, textToDom);
    },
    updateOutput: function () {
      console.log("NDisplay =");
      console.log(this.nDisplay);
    },
  },
  watch: {
    nDisplay: {
      deep: true,
      handler: function (newValue) {
        console.log("Selected users changed", newValue);
        if (newValue == false) {
          this.url = this.streamUrl +"?topic=/output/image_raw&type=ros_compressed";
          this.$refs.displayImage.src = this.url;
          this.$refs.displayImageFull.src = this.url;
        } else if (newValue == true) {
          this.url = this.streamUrl +"?topic=/output/image_detected&type=ros_compressed";
          this.$refs.displayImage.src = this.url;
          this.$refs.displayImageFull.src = this.url;
        }
      },
    },
    s_result: function (val) {
      console.log("watch = ");
      this.result += val;
      //console.log(val.toString(16))
      var ss = val.replace(/(<([^>]+)>)/gi, "");

      //const ss = strippedString.replace(/[\n\r\t]/g,);
      var base = 16;

      /*console.log(ss.split('').map(function (c) {
                return c.charCodeAt(0);
            }))
            console.log(ss)*/
      if (ss.replace(/(\r\n|\n|\r)/gm, "") === "DONE") {
        console.log("Finished");
      }
    },
  },

  mounted() {
    const base_blocks = `<xml ref="toolbox" style="display: none">
      <category name="Logic" colour="%{BKY_LOGIC_HUE}">
      <category name="If">
        <block type="controls_if"></block>
        <block type="controls_if">
          <mutation else="1"></mutation>
        </block>
        <block type="controls_if">
          <mutation elseif="1" else="1"></mutation>
        </block>
      </category>
      <category name="Boolean" colour="%{BKY_LOGIC_HUE}">
        <block type="logic_compare"></block>
        <block type="logic_operation"></block>
        <block type="logic_negate"></block>
        <block type="logic_boolean"></block>
        <block type="logic_null"></block>
        <block type="logic_ternary"></block>
      </category>
    </category>
    <category name="Loops" colour="%{BKY_LOOPS_HUE}">
      <block type="controls_repeat_ext">
        <value name="TIMES">
          <block type="math_number">
            <field name="NUM">10</field>
          </block>
        </value>
      </block>
      <block type="controls_whileUntil"></block>
      <block type="controls_for">
        <field name="VAR">i</field>
        <value name="FROM">
          <block type="math_number">
            <field name="NUM">1</field>
          </block>
        </value>
        <value name="TO">
          <block type="math_number">
            <field name="NUM">10</field>
          </block>
        </value>
        <value name="BY">
          <block type="math_number">
            <field name="NUM">1</field>
          </block>
        </value>
      </block>
      <block type="controls_forEach"></block>
      <block type="controls_flow_statements"></block>
    </category>
    <category name="Math" colour="%{BKY_MATH_HUE}">
      <block type="math_number">
        <field name="NUM">123</field>
      </block>
      <block type="math_arithmetic"></block>
      <block type="math_single"></block>
      <block type="math_trig"></block>
      <block type="math_constant"></block>
      <block type="math_number_property"></block>
      <block type="math_round"></block>
      <block type="math_on_list"></block>
      <block type="math_modulo"></block>
      <block type="math_constrain">
        <value name="LOW">
          <block type="math_number">
            <field name="NUM">1</field>
          </block>
        </value>
        <value name="HIGH">
          <block type="math_number">
            <field name="NUM">100</field>
          </block>
        </value>
      </block>
      <block type="math_random_int">
        <value name="FROM">
          <block type="math_number">
            <field name="NUM">1</field>
          </block>
        </value>
        <value name="TO">
          <block type="math_number">
            <field name="NUM">100</field>
          </block>
        </value>
      </block>
      <block type="math_random_float"></block>
      <block type="math_atan2"></block>
    </category>
    <category name="Text" colour="%{BKY_TEXTS_HUE}">
    <block type="text"></block>
    <block type="text_length"></block>
    <block type="text_print"></block>
</category>
    <category name="Lists" colour="%{BKY_LISTS_HUE}">
      <block type="lists_create_empty"></block>
      <block type="lists_create_with"></block>
      <block type="lists_repeat">
        <value name="NUM">
          <block type="math_number">
            <field name="NUM">5</field>
          </block>
        </value>
      </block>
      <block type="lists_length"></block>
      <block type="lists_isEmpty"></block>
      <block type="lists_indexOf"></block>
      <block type="lists_getIndex"></block>
      <block type="lists_setIndex"></block>
    </category>
    <sep></sep>
    <category name="Variables" custom="VARIABLE" colour="%{BKY_VARIABLES_HUE}">
    </category>
    <category name="Functions" custom="PROCEDURE" colour="%{BKY_PROCEDURES_HUE}">
    </category>
    <sep></sep>
      
      <category name="KidBright AI" colour="%{BKY_VARIABLES_HUE}">
      <block type="init_ros_node"></block>
      <block type="start_object_detector"></block>
      <block type="start_image_classification"></block>
      <block type="start_wake_word_detector"></block>
      <block type="rospy_loop"></block>
      <block type="get_objects"></block>
      <block type="get_classes"></block>
      <block type="get_sound"></block>

      <block type="get_object_attr"></block>
      <block type="set_velocity"></block>
      <block type="delay"></block>
  </category>

  </xml>`;

    Blockly.Python["start_object_detector"] = function (block) {
      var cc =
        "import rosnode\nimport subprocess\nimport time\nimport os\nros_nodes = rosnode.get_node_names()\nif not '/image_feature' in ros_nodes:\n";
      cc =
        cc +
        "\tcommand='rosrun kidbright_tpu tpu_detect.py " +
        process.env.VUE_APP_ROOT +
        "/" +
        this.$store.getters.getProjectDir +
        "'\n";
      cc =
        cc +
        "\tprocess = subprocess.Popen(command, shell=True, stdout=subprocess.PIPE)\n\ttime.sleep(10) \n";

      return cc;
    }.bind(this);

    Blockly.Python["start_image_classification"] = function (block) {
      var cc =
        "import rosnode\nimport subprocess\nimport time\nimport os\nros_nodes = rosnode.get_node_names()\nif not '/image_class' in ros_nodes:\n";
      cc =
        cc +
        "\tcommand='rosrun kidbright_tpu tpu_classify.py " +
        process.env.VUE_APP_ROOT +
        "/" +
        this.$store.getters.getProjectDir +
        "'\n";
      cc =
        cc +
        "\tprocess = subprocess.Popen(command, shell=True, stdout=subprocess.PIPE)\n\ttime.sleep(10) \n";

      return cc;
    }.bind(this);

    Blockly.Python["start_wake_word_detector"] = function (block) {
      var dur = this.$store.getters.getProjDescription
      //console.log("==========> Duraion is =======>");
      //console.log(dur.Duration*4);
      var nf = dur.Duration*4
      var cc =
        "import rosnode\nimport subprocess\nimport time\nimport os\nros_nodes = rosnode.get_node_names()\nif not '/wake_class_wait' in ros_nodes:\n";
      cc =
        cc +
        "\tcommand='rosrun kidbright_tpu wakeword_classify.py " +
        "_terminate:=False "  +
        "_model:=" +
        process.env.VUE_APP_ROOT +
        "/" +
        this.$store.getters.getProjectDir +
        "/audios/model.h5 " + 
        "label_file:=" +
        process.env.VUE_APP_ROOT +
        "/" +
        this.$store.getters.getProjectDir +
        "/audios/label_map.pkl " +
        "_nframe:=" + 
        nf.toString() +
        "'\n";
      cc =
        cc +
        "\tprocess = subprocess.Popen(command, shell=True, stdout=subprocess.PIPE)\n\ttime.sleep(10) \n";

      return cc;
    }.bind(this);

    Blockly.Python["init_ros_node"] = function (block) {
      var code =
        "from geometry_msgs.msg import Twist\nimport rospy\nimport time\nrospy.init_node('get_center', anonymous=True)\nvelocity_publisher = rospy.Publisher('/cmd_vel', Twist, queue_size=1)\nvel_msg = Twist()\n";

      code =
        code +
        "import roslib\nimport rospy\nfrom kidbright_tpu.msg import tpu_object\nfrom kidbright_tpu.msg import tpu_objects\nfrom std_msgs.msg import String\n";
      return code;
    };

    Blockly.Blocks["get_objects"] = {
      init: function () {
        this.appendDummyInput().appendField("get objects");
        this.setOutput(true, null);
        this.setColour(230);
        this.setTooltip("");
        this.setHelpUrl("");
      },
    };

    Blockly.Blocks["get_classes"] = {
      init: function () {
        this.appendDummyInput().appendField("get classes");
        this.setOutput(true, null);
        this.setColour(230);
        this.setTooltip("");
        this.setHelpUrl("");
      },
    };

    Blockly.Blocks["get_sound"] = {
      init: function () {
        this.appendDummyInput().appendField("get sound");
        this.setOutput(true, null);
        this.setColour(230);
        this.setTooltip("");
        this.setHelpUrl("");
      },
    };

    Blockly.Blocks["sumorobot_opponent"] = {
      init: function () {
        this.setColour("#0099E6");
        this.appendDummyInput().appendField("opponent");
        this.setOutput(true, "Boolean");
      },
    };

    Blockly.Blocks["start_object_detector"] = {
      init: function () {
        this.appendDummyInput().appendField("Start object detector");
        this.setPreviousStatement(true, null);
        this.setNextStatement(true, null);
        this.setColour(230);
        this.setTooltip("");
        this.setHelpUrl("");
      },
    };

    Blockly.Blocks["start_image_classification"] = {
      init: function () {
        this.appendDummyInput().appendField("Start image classification");
        this.setPreviousStatement(true, null);
        this.setNextStatement(true, null);
        this.setColour(230);
        this.setTooltip("");
        this.setHelpUrl("");
      },
    };

    Blockly.Blocks["start_wake_word_detector"] = {
      init: function () {
        this.appendDummyInput().appendField("Start wake word detector");
        this.setPreviousStatement(true, null);
        this.setNextStatement(true, null);
        this.setColour(230);
        this.setTooltip("");
        this.setHelpUrl("");
      },
    };

    Blockly.Blocks["init_ros_node"] = {
      init: function () {
        this.appendDummyInput().appendField("ROS node initialization");
        this.setPreviousStatement(true, null);
        this.setNextStatement(true, null);
        this.setColour(230);
        this.setTooltip("");
        this.setHelpUrl("");
      },
    };

    Blockly.Blocks["rospy_loop"] = {
      init: function () {
        this.appendDummyInput().appendField("ROS LOOP");
        this.appendStatementInput("DO")
          .setCheck(null)
          .setAlign(Blockly.ALIGN_RIGHT);
        this.setPreviousStatement(true, null);
        this.setNextStatement(true, null);
        this.setColour(230);
        this.setTooltip("");
        this.setHelpUrl("");
      },
    };

    Blockly.Blocks["get_object_attr"] = {
      init: function () {
        this.appendValueInput("VAR")
          .setCheck(null)
          .appendField("get")
          .appendField(
            new Blockly.FieldDropdown([
              ["cx", "cx"],
              ["cy", "cy"],
              ["width", "width"],
              ["height", "height"],
              ["label", "label"],
            ]),
            "DATA_FIELD"
          )
          .appendField(" from");
        this.setOutput(true, null);
        this.setColour(230);
        this.setTooltip("");
        this.setHelpUrl("");
      },
    };

    Blockly.Blocks["set_velocity"] = {
      init: function () {
        this.appendDummyInput().appendField("move with ");
        this.appendDummyInput()
          .appendField("linear velocity")
          .appendField(new Blockly.FieldNumber(0), "LINEAR");
        this.appendDummyInput()
          .appendField("angular velocity")
          .appendField(new Blockly.FieldNumber(0), "ANGULAR");
        this.setInputsInline(true);
        this.setPreviousStatement(true, null);
        this.setNextStatement(true, null);
        this.setColour(230);
        this.setTooltip("");
        this.setHelpUrl("");
      },
    };

    Blockly.Blocks["delay"] = {
      init: function () {
        this.appendDummyInput().appendField("delay");
        this.appendValueInput("NAME").setCheck(null);
        this.setInputsInline(true);
        this.setPreviousStatement(true, null);
        this.setNextStatement(true, null);
        this.setColour(230);
        this.setTooltip("");
        this.setHelpUrl("");
      },
    };

    Blockly.Python["set_velocity"] = function (block) {
      var number_linear = block.getFieldValue("LINEAR");
      var number_angular = block.getFieldValue("ANGULAR");
      var code =
        "vel_msg.linear.y = 0\nvel_msg.linear.z = 0\nvel_msg.angular.x = 0\nvel_msg.angular.y = 0\n";
      code =
        code +
        "vel_msg.linear.x = " +
        number_linear +
        "\n" +
        "vel_msg.angular.z = " +
        number_angular +
        "\n";
      code = code + "velocity_publisher.publish(vel_msg)\n";

      return code;
    };

    Blockly.Python["get_object_attr"] = function (block) {
      var dropdown_data_field = block.getFieldValue("DATA_FIELD");
      var value_var = Blockly.Python.valueToCode(
        block,
        "VAR",
        Blockly.Python.ORDER_ATOMIC
      );
      // TODO: Assemble Python into code variable.
      console.log(block.getFieldValue("DATA_FIELD"));
      var code = value_var + "." + block.getFieldValue("DATA_FIELD");
      // TODO: Change ORDER_NONE to the correct strength.
      return [code, Blockly.Python.ORDER_NONE];
    };

    Blockly.Python["rospy_loop"] = function (block) {
      var statements_name = Blockly.Python.statementToCode(block, "NAME");
      // TODO: Assemble Python into code variable.
      var branch = Blockly.Python.statementToCode(block, "DO");
      branch = Blockly.Python.addLoopTrap(branch, block) || Blockly.Python.PASS;
      var code = "while not rospy.is_shutdown():\n" + branch;
      return code;
    };

    Blockly.Python["get_objects"] = function (block) {
      var code =
        "rospy.wait_for_message('/tpu_objects', tpu_objects, timeout=4).tpu_objects";
      // TODO: Change ORDER_NONE to the correct strength.
      return [code, Blockly.Python.ORDER_NONE];
    };

    Blockly.Python["get_classes"] = function (block) {
      // TODO: Assemble Python into code variable.
      var code =
        "rospy.wait_for_message('/tpu_objects', tpu_objects, timeout=4).tpu_objects";
      // TODO: Change ORDER_NONE to the correct strength.
      return [code, Blockly.Python.ORDER_NONE];
    };

    Blockly.Python["get_sound"] = function (block) {
      // TODO: Assemble Python into code variable.
      var code =
        "rospy.wait_for_message('/inference', String, timeout=4).data";
      // TODO: Change ORDER_NONE to the correct strength.
      return [code, Blockly.Python.ORDER_NONE];
    };

    Blockly.Python["delay"] = function (block) {
      var value_name = Blockly.Python.valueToCode(
        block,
        "NAME",
        Blockly.Python.ORDER_ATOMIC
      );
      // TODO: Assemble Python into code variable.
      var code = "time.sleep(" + value_name + ")\n";
      return code;
    };

    setTimeout(() => {
      //alert("Hello blockly")
      var tt = {};
      tt.toolbox = base_blocks;
      tt.scrollbars = true;
      tt.css = true;
      tt.zoom = {
        controls: true,
        wheel: true,
        startScale: 1.0,
        maxScale: 4,
        minScale: 0.25,
        scaleSpeed: 1.1,
      };

      var blckDiv = this.$refs["blocklyDiv"];
      this.blockly_woakspace = Blockly.inject(this.$refs["blocklyDiv"], tt);
      //>>>>>>>>>>>> this.$store.commit("setBlocklyWorkspace", this.blockly_woakspace);
      console.log("Injection running ***********************************");
      //console.log(this.blockly_woakspace)
      this.loaded = true;
    }, 500);
  },
  created() {
    this.unsubscribe = this.$store.subscribe((mutation, state) => {
      if (mutation.type === "setBlocklyXml") {
        setTimeout(() => {
          console.log("updating xml !!!!!!!!!");
          //console.log(state.blockly_xml)
          this.blockly_xml = state.blockly_xml;
          // console.log('blockly_xml == ',this.blockly_xml)
          //this.blockly_woakspace = state.blockly_xml
          this.blockly_woakspace.clear();
          let textToDom = Blockly.Xml.textToDom(this.blockly_xml);
          Blockly.Xml.domToWorkspace(this.blockly_woakspace, textToDom);
        }, 500);
      }
    });
  },
  beforeDestroy() {
    this.unsubscribe();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->

<style lang="scss" scoped>
$primary-color: #007e4e;
$black: #333;
$yellow: #fff7d6;
$grey: #eeeeee;

* {
  margin: 0;
  padding: 0;
  outline: none;
  box-sizing: border-box;
}

ul {
  list-style: none;
  padding: 0;
}

.op-btn {
  transition: opacity 0.3s ease-in;
  cursor: pointer;

  &:hover {
    opacity: 0.7;
  }
}

.main-panel {
  width: calc(100% - 300px);
}

.blockly-module {
  display: flex;
  position: relative;
  top: 0;
  left: 0;
  height: 100%;
  width: 100%;

  .blockly {
    height: 100%;
    width: 100%;
  }

  .side-panel {
    padding: 15px;
    position: absolute;
    top: 0;
    bottom: 0;
    right: 0;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    width: 300px;
    text-align: end;

    .btn-run {
      img {
        width: 100px;
      }
    }

    .btn-stop {
      img {
        width: 100px;
      }
    }

    .display-panel {
      border-radius: 8px;
      background-color: #333;
      overflow: hidden;
      border: #adb5bd solid 1px;

      .display-image {
        img {
          min-height: 180px;
          height: 180px;
          width: 100%;
          object-fit: cover;
        }
      }

      .control {
        background-color: #ffffff;
        width: 100%;
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 10px;
        text-transform: uppercase;

        .check {
          padding-left: 36px;
          font-size: 0.7rem;
          color: #222222;
          display: flex;
          align-items: center;
        }
      }
    }

    .next {
      height: 50px;
      background: #ffffff 0% 0% no-repeat padding-box;
      border-radius: 38px;
      display: flex;
      justify-content: center;
      align-items: center;
      position: relative;
      margin-top: 15px;

      span {
        color: $primary-color;
        font-size: 1.5rem;
        font-weight: 800;

        &.ico {
          position: absolute;
          top: 7px;
          right: 18px;
        }
      }
    }
  }
}
#expanded-camera {
  .result {
    height: 20vh;
    .bg-secondary {
      height: 100%;
    }
  }
  .display-image {
    background-color: #333 !important;
    img.realtime-image {
      width: 100%;
      height: 55vh;
      object-fit: contain;
    }
  }
  .control {
    background-color: #ffffff;
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 10px;
    text-transform: uppercase;

    .check {
      padding-left: 36px;
      font-size: 0.7rem;
      color: #222222;
      display: flex;
      align-items: center;
    }
  }
}
</style>
<style lang="scss" scoped>
button {
  color: unset;
  border: unset;
  background-color: unset;
  text-align: left;
  position: relative;

  &::after {
    position: absolute;
    top: 17px;
    right: 15px;
  }
}

.custom-control-label::before {
  top: 0 !important;
}

.custom-control-label::after {
  top: 2px !important;
}

.custom-control-input:checked ~ .custom-control-label::before {
  color: #333;
  border-color: #333;
  background-color: #333;
}

.train-pgr {
  border: none !important;
}

.scroll-box {
  height: 200px;
  overflow-y: scroll;
  text-align: left;
  padding: 20px !important;
  background-color: #333 !important;
  display: flex;
  flex-direction: column-reverse;
}

//.modal-footer {
//    display: none !important;
//}
</style>
