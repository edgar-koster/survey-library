<template>
  <div
    v-if="question.isVisible"
    :class="question.cssClasses.panel.container"
    :style="rootStyle"
  >
    <h4 v-show="hasTitle" :class="getTitleStyle()" v-on:click="changeExpanded">
      <span v-if="question.isRequireTextOnStart" :class="requiredTextCss">{{
        question.requiredText
      }}</span>
      <span
        v-if="question.no"
        style="position: static"
        :class="question.cssClasses.number"
        >{{ question.no }}</span
      >
      <span v-if="question.isRequireTextBeforeTitle" :class="requiredTextCss">{{
        question.requiredText
      }}</span>
      <survey-string :locString="question.locTitle" />
      <span v-if="question.isRequireTextAfterTitle" :class="requiredTextCss">{{
        question.requiredText
      }}</span>
      <span
        v-show="showIcon"
        :class="iconCss"
        v-on:keyup.enter="changeExpanded"
        tabindex="0"
        v-bind:aria-expanded="isCollapsed ? 'false' : 'true'"
        :aria-controls="!isCollapsed ? question.contentId : null"
      ></span>
    </h4>
    <div :class="question.cssClasses.panel.description">
      <survey-string :locString="question.locDescription" />
    </div>
    <survey-errors :question="question" />
    <div
      :id="question.contentId"
      :style="{ paddingLeft: question.innerPaddingLeft }"
      v-show="!isCollapsed"
      :class="question.cssClasses.panel.content"
    >
      <div
        v-for="(row, index) in rows"
        :key="question.id + '_' + index"
        v-if="row.visible"
        :class="css.row"
      >
        <survey-row :row="row" :survey="survey" :css="css"></survey-row>
      </div>
      <div
        v-if="question.hasEditButton"
        :class="question.cssClasses.panel.footer"
      >
        <input
          type="button"
          :value="survey.editText"
          :class="survey.cssNavigationEdit"
          @click="cancelPreview"
        />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import { Component, Prop } from "vue-property-decorator";
import { PanelModelBase, PanelModel, QuestionRowModel } from "../panel";
import { ISurvey, Base } from "../base";
import { BaseVue } from "./base";

@Component
export class Panel extends BaseVue {
  @Prop question: PanelModel;
  @Prop isEditMode: Boolean;
  @Prop css: any;
  
  private isCollapsed: boolean = false;

  protected getModel(): Base {
    return this.question;
  }
  protected onMounted() {
    if (this.question.survey) {
      this.question.survey.afterRenderPanel(this.question, this.$el);
    }
    this.isCollapsed = this.question.isCollapsed;

    this.question.stateChangedCallback = () => {
      this.isCollapsed = this.question.isCollapsed;
    }
  }
  beforeDestroy() {
    this.question.stateChangedCallback = null;
  }
  get rootStyle() {
    var result = {};
    if (this.question.renderWidth) {
      (<any>result)["flexBasis"] = this.question.renderWidth;
      (<any>result)["flexGrow"] = 1;
      (<any>result)["flexShrink"] = 1;
      (<any>result)["width"] = this.question.renderWidth;
      (<any>result)["minWidth"] = this.question.minWidth;
      (<any>result)["maxWidth"] = this.question.maxWidth;
    }
    return result;
  }
  get showIcon() {
    return (
      this.question.isExpanded || this.question.isCollapsed
    );
  }
  get rows() {
    return this.question.rows;
  }
  get hasTitle() {
    return this.question.title.length > 0;
  }
  get survey() {
    return this.question.survey;
  }
  get iconCss() {
    var result = this.css.panel.icon;
    if (!this.isCollapsed) result += " " + this.css.panel.iconExpanded;
    return result;
  }
  changeExpanded() {
    this.question.toggleState();
  }
  cancelPreview() {
    this.question.cancelPreview();
  }
  getTitleStyle() {
    var result = this.css.panel.title;
    if (this.question.isCollapsed || this.question.isExpanded) {
      result += " " + this.css.panel.titleExpandable;
    }
    if (this.question.containsErrors) {
      result += " " + this.question.cssClasses.panel.titleOnError;
    }
    return result;
  }

  get requiredTextCss() {
    return (
      this.question.cssClasses.requiredText ||
      this.question.cssClasses.panel.requiredText
    );
  }
}
Vue.component("survey-panel", Panel);
export default Panel;
</script>
