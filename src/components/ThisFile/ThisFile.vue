<template>
  <div :class="getContainerClass">
    <label v-if="!getRemoveLabel" class="label has-text-left">{{
      label
    }}</label>
    <div :class="getClasses">
      <label class="file-label file-label-container">
        <input
          :id="id"
          class="file-input"
          type="file"
          :disabled="isUploading"
          ref="files"
          @change="handleFilesUpload()"
          :accept="accept"
        />
        <span class="file-content is-size-6">
          <span class="file-icon">
            <this-icon
              :preserve-defaults="!overrideDefaults"
              :icon="$thisvui.icons.upload"
            ></this-icon>
          </span>
          <span class="file-label">
            <p v-if="isInitialUpload">
              Choose your file(s)
            </p>
            <p v-if="isUploading">Uploading {{ fileCount }} files...</p>
          </span>
          <span class="file-name" v-if="fileHasName"> </span>
        </span>
      </label>
    </div>
    <div class="preview" v-if="showPreview">
      <h2 v-if="isUploadSuccess" class="is-size-6">
        Uploaded {{ files.length }} file(s) successfully.
      </h2>
      <this-buttons>
        <button @click="reset()" :class="`button ${clearClass}`">
          <this-icon :icon="clearIcon"></this-icon><span>{{ clearText }}</span>
        </button>
        <button
          @click="submitFiles()"
          v-if="!isAutoSubmit && showSubmitButton"
          :class="`button ${submitClass}`"
        >
          <this-icon :icon="submitIcon"></this-icon
          ><span>{{ submitText }}</span>
        </button>
      </this-buttons>
      <this-thumbnails>
        <this-thumbnail v-for="(file, key) in files" :key="key">
          <img class="image is-128x128" v-bind:ref="`image${parseInt(key)}`" />
          <h1>{{ file.name }}</h1>
        </this-thumbnail>
      </this-thumbnails>
    </div>
    <this-box v-if="isUploadFailed">
      <h2>Uploaded failed.</h2>
      <p>
        <a href="javascript:void(0)" @click="reset()">Try again</a>
      </p>
      <pre>{{ uploadError }}</pre>
    </this-box>
  </div>
</template>

<script>
import input from "../../mixins/input";
import alignment from "../../mixins/alignment";
import icons from "../../mixins/icons";
import CssArchitect from "../../utils/css-architect";
import ThisBox from "../ThisBox/ThisBox";
import ThisIcon from "../ThisIcon/ThisIcon";
import ThisThumbnails from "../ThisThumbnail/ThisThumbnails";
import ThisThumbnail from "../ThisThumbnail/ThisThumbnail";
import ThisButtons from "../ThisAction/ThisButtons";
import ThisAction from "../ThisAction/ThisAction";

const STATUS_INITIAL = 0;
const STATUS_SAVING = 1;
const STATUS_SUCCESS = 2;
const STATUS_FAILED = 3;

export default {
  name: "ThisFile",
  components: {
    ThisAction,
    ThisButtons,
    ThisThumbnail,
    ThisThumbnails,
    ThisIcon,
    ThisBox
  },
  mixins: [input, alignment, icons],
  props: {
    accept: {
      type: String,
      default: "*"
    },
    onUpload: {
      type: Function
    },
    hasName: {
      type: [String, Boolean]
    },
    multiple: {
      type: [Boolean, String],
      default: true
    },
    showSubmit: {
      type: [Boolean, String],
      default: true
    },
    autoSubmit: {
      type: [Boolean, String],
      default: false
    },
    isBoxed: {
      type: [String, Boolean]
    },
    targetClass: {
      type: String
    },
    chooseText: {
      type: String,
      default: "Choose your file(s)"
    },
    submitText: {
      type: String,
      default: "Submit"
    },
    submitIcon: {
      type: String,
      default: function() {
        return this.$thisvui.icons.upload;
      }
    },
    submitClass: {
      type: String,
      default: "is-success"
    },
    clearText: {
      type: String,
      default: "Clear"
    },
    clearIcon: {
      type: String,
      default: function() {
        return this.$thisvui.icons.remove;
      }
    },
    clearClass: {
      type: String,
      default: "is-danger"
    }
  },
  data() {
    return {
      files: [],
      uploadError: null,
      currentStatus: null,
      showPreview: false
    };
  },
  computed: {
    /**
     * Dynamically build the css classes for the target element
     * @returns { A String with the chained css classes }
     */
    getClasses: function() {
      const cssArchitect = new CssArchitect("file");
      cssArchitect.addClass(this.getSyntaxModifiers);
      cssArchitect.addClass(this.getColorsModifiers);
      cssArchitect.addClass(this.getAlignmentModifiers);
      cssArchitect.addClass(this.targetClass);
      cssArchitect.addClass("has-name", this.fileHasName);
      cssArchitect.addClass("is-boxed", this.getBoolean(this.isBoxed));
      return cssArchitect.getClasses();
    },
    fileHasName() {
      return this.getBoolean(this.hasName);
    },
    showSubmitButton() {
      return this.getBoolean(this.showSubmit);
    },
    isAutoSubmit() {
      return this.getBoolean(this.autoSubmit);
    },
    isInitialUpload() {
      return this.currentStatus === STATUS_INITIAL;
    },
    isUploading() {
      return this.currentStatus === STATUS_SAVING;
    },
    isUploadSuccess() {
      return this.currentStatus === STATUS_SUCCESS;
    },
    isUploadFailed() {
      return this.currentStatus === STATUS_FAILED;
    }
  },
  methods: {
    /**
     * Resets data props that set the state
     */
    reset() {
      this.currentStatus = STATUS_INITIAL;
      this.files = [];
      this.uploadError = null;
      this.showPreview = false;
    },
    /**
     * Builds the form data and execute the onUpload function
     */
    async submitFiles() {
      this.currentStatus = STATUS_SAVING;
      if (this.onUpload) {
        try {
          let formData = new FormData();

          // Iterate over any file sent over appending the files
          // to the form data.
          for (var i = 0; i < this.files.length; i++) {
            let file = this.files[i];

            formData.append("files[" + i + "]", file);
          }
          await this.onUpload(formData);
          this.$emit("uploaded");
        } catch (e) {
          console.error(e);
          this.$emit("failed");
          this.currentStatus = STATUS_FAILED;
        }
      } else {
        throw new Error(
          "You should provide an upload function trough the onUpload prop"
        );
      }
    },
    /**
     * Handle the files upload process
     */
    handleFilesUpload() {
      //  Get the uploaded files from the input.
      let uploadedFiles = this.$refs.files.files;

      // Adds the uploaded file to the files array
      for (let i = 0; i < uploadedFiles.length; i++) {
        this.files.push(uploadedFiles[i]);
      }

      // Generate image previews for the uploaded files
      this.showPreview = true;
      this.getImagePreviews();
      // If autoSubmit is active we submit the files automatically
      if (this.autoSubmit) {
        this.submitFiles();
      }
    },

    /**
     * Gets the preview image for the file.
     */
    getImagePreviews() {
      // Iterate over all of the files and generate an image preview for each one.
      for (let i = 0; i < this.files.length; i++) {
        // Ensure the file is an image file
        if (/\.(jpe?g|png|gif)$/i.test(this.files[i].name)) {
          // Creates a new FileReader object
          let reader = new FileReader();

          // Add an event listener for when the file has been loaded
          // to update the src on the file preview.
          reader.addEventListener(
            "load",
            function() {
              this.$refs[`image${parseInt(i)}`][0].src = reader.result;
            }.bind(this),
            false
          );

          // Read the data for the file in through the reader. When it has
          // been loaded, we listen to the event propagated and set the image
          // src to what was loaded from the reader.
          reader.readAsDataURL(this.files[i]);
        }
      }
    }
  },
  mounted() {
    this.reset();
  }
};
</script>
