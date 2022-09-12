<template>
  <div>
    <div id="workArea">
      Input Area
      <textarea v-model="initalArea" @input="splitFirst"></textarea>
      <div style="display: inline">
        <br />
        Split
        <button type="button" @click="toggleVisible">Show/Hide Split</button>
      </div>
      <textarea id="firstSplit" v-model="fistSplitArea"></textarea>
    </div>
    <div id="paddingDiv"></div>
    <div id="resultArea">
      <textarea v-model="resultArea" disabled></textarea>
      Results Area
      <textarea v-model="assocResultArea" disabled></textarea>
    </div>
    <div>
      <button @click="download">Download</button>
    </div>
  </div>
</template>

<script>
import words from "~~/module/words";

export default {
  data() {
    return {
      initalArea: "",
      fistSplitArea: "",
      resultArea: "",
      assocResultArea: "",
    };
  },
  methods: {
    splitFirst(event) {
      // Splits the TextArea at new lines and removes whitespace
      // Calls the countRepeats method

      this.fistSplitArea = this.initalArea
        .split(";")
        .map((item) => item.trim())
        .join("\n");
      this.countRepeats();
    },
    countRepeats(event) {
      // Counts the number of times a word is repeated in the second split textarea

      this.resultArea = "";
      let finalCount = [];
      const items = this.fistSplitArea.split("\n");

      // Loops through the items
      // Checks if the word is already in the final Count array
      // Creates a new object with the word and the number of times it is repeated
      // Sets the values of the new object
      // Pushes the new object to the finalCount array
      items.forEach((item) => {
        if (
          !finalCount.find((o) => o.word === item) &&
          item != "" &&
          item != " "
        ) {
          let newOb = { word: null, count: null, assoc: "No Assoc\n" };
          newOb.word = item;
          newOb.count = 0;
          finalCount.push(newOb);
        }
      });

      // Loops through the items array
      // Loops through the finalCount array
      // Checks the finalItem.word against the item
      // If found increments the finalItem.count
      // If there is no associated word ->
      // Loops through the words array
      // Checks if the word is the same as the finalItem.word
      // If found sets the finalItem.assoc to the word

      items.forEach((item) => {
        finalCount.forEach((finalItem) => {
          if (finalItem.word === item) {
            finalItem.count++;
            if (finalItem.assoc === "No Assoc\n") {
              words().forEach((element) => {
                if (element.word === item) {
                  finalItem.assoc = element.assoc + "\n";
                }
              });
            }
          }
        });
      });
      finalCount.pop(); // remove last item it is empty

      // Sorts the finalCount array by word count
      finalCount.sort((a, b) => {
        return b.count - a.count;
      });

      // Loops through the finalCount array
      // Counts the associated words
      let assocCount = [];
      finalCount.forEach((item) => {
        let returnValue = "";
        try {
          returnValue = item.assoc.replace("\n", "") + ", " + item.count;
        } catch (error) {
          returnValue = item.assoc + ", " + item.count;
        }

        assocCount.push(returnValue);
      });

      // Sets the resultArea and assocResultArea to the finalCount array
      this.resultArea = this.objToString(finalCount);
      this.assocResultArea = assocCount.join("\n");
      console.log(assocCount);
    },
    objToString(obj) {
      // Converts the obj to a string
      // Formats it for output
      let str = "";
      obj.forEach((item) => {
        try {
          return (str +=
            item.word.replace(",", ";") +
            ", " +
            item.count +
            ", " +
            item.assoc.replace(",", ";"));
        } catch (error) {
          return (str += item.word + ", " + item.count + ", " + item.assoc);
        }
      });
      return str;
    },
    download() {
      // Downloads the resultArea and assocResultArea to a csv file
      // I don't know how this works, GitHub Copilot wrote it lol
      let date = new Date();
      let dateString =
        date.getFullYear() + "-" + (date.getMonth() + 1) + "-" + date.getDate();
      let timeString =
        date.getHours() + "-" + date.getMinutes() + "-" + date.getSeconds();
      let text = this.resultArea + "\n" + this.assocResultArea;
      let blob = new Blob([text], { type: "text/plain;charset=utf-8" });
      this.saveAs(blob, `${dateString + timeString}.csv`);
    },
    saveAs(blob, fileName) {
      let a = document.createElement("a");
      a.href = URL.createObjectURL(blob);
      a.download = fileName;
      a.click();
    },
    toggleVisible(e) {
      let elSelect = document.getElementById("firstSplit");
      if (elSelect.style.display != "block") {
        elSelect.style.display = "block";
      } else {
        elSelect.style.display = "none";
      }
    },
  },
};
</script>

<style scoped>
div {
  display: grid;
  grid-template-columns: 1fr;
}

#firstSplit {
  display: none;
}

#paddingDiv {
  padding: 10px;
}

#workArea textarea {
  height: 100px;
}

#resultArea textarea {
  height: 100px;
}

#workArea textarea,
#resultArea textarea {
  resize: none;
}
</style>
