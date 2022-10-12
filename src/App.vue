<template>
  <nav>
    <ul>
      <li id="header_title"><div>Timeloops</div></li>
      <li
        v-for="(tab, index) in tabs" :key="index"
      >
        <div
          :class="{active: curTab == tab}"
          @click="curTab = tab"
        > {{ tab }}</div>
      </li>
    </ul>
  </nav>
  
    <!-- <div id="edit_todo_toggle">
      <label class="toggle">
        <input class="toggle-checkbox" type="checkbox"
          v-model="editMode"
        >
        <div class="toggle-switch"></div>
        <span class="toggle-label">Edit mode</span>
      </label>
    </div> -->
  <div id="page_wrapper">
    <div v-if="curTab == 'play'" class="template_ready">
      <div class="header">
        <div class="header_part">
          <div class="title">
            {{ template.title }}
          </div>
          <div 
            class="play_button"
            @click="toogleLoop"
          >
            <i v-if="time.loopIsRunning" class="fa fa-pause" aria-hidden="true"></i>
            <i v-else class="fa fa-play" aria-hidden="true"></i>
          </div>
        </div>
        <div class="timeline">
          <div 
            :style="{'width':time.widthPercentage}"
            class="progress_bar"
          ></div>
        </div>
      </div>
      <div class="parts">
        <div
        class="part"
        v-for="(part, index) in template.parts" :key="index"
        :style="{'background-color':part.color}"
        >
          <div class="part_wrapper">
            <div class="part_content">
              <div class="icons">
                <span class="icon_part part_num">
                  {{index + 1}}.
                </span>
                <span class="icon_part">
                  <i 
                    class="fa fa-step-forward" aria-hidden="true"
                    :class="{active: part.pauseOnFinish}"
                  ></i>
                </span>
                <span class="icon_part">
                  <i 
                    class="fa fa-refresh" aria-hidden="true"
                    :class="{active: part.repeat, span_margin: part.repeat}"
                  ></i>
                  <span 
                    v-if="part.repeat"
                    class="text"
                  >{{time.parts[index].loopsToDo - time.parts[index].loopsDone}}</span>
                </span>
                <span class="icon_part">
                  <i 
                    class="fa fa-volume-up" aria-hidden="true"
                    :class="{active: part.playAlarm}"
                  ></i>
                </span>
              </div>
              <div class="timeline_holder">
                <div class="timeline">
                  <div 
                    :style="{'width':time.parts[index].widthPercentage}"
                    class="progress_bar"
                  ></div>
                </div>
                <div 
                  class="play_button"
                  @click="handlePartClick(index)"
                >
                  <div>
                    <i v-if="time.curIndex == index && time.loopIsRunning" class="fa fa-pause" aria-hidden="true"></i>
                    <i v-else class="fa fa-play" aria-hidden="true"></i>
                  </div>
                </div>
              </div>
              <div class="countdown_time">
                <div>
                  {{time.parts[index].timeLeft}}
                </div>
              </div>
            </div>
          </div>
          <div 
            class="part_title"
          >{{ part.title }}</div>
        </div>
      </div>
    </div>  
    <div v-if="curTab == 'editor'" class="template">
      <div class="header">
        <div>
          <input
            class="title"  
            v-model="template.title"
            type="text"
            placeholder="template title"
          >
          <!-- <div class="divider"></div>
          <label>
            <input type="checkbox" v-model="template.looped"/>
            Loop
          </label>
          <span v-if="!template.looped">
            <br>
            &emsp;Repeat
            <input
              v-model="template.repeatTimes"
              type="number"
            > times
          </span> -->
        </div>
      </div>
      <div class="parts">
        <div
          class="part"
          v-for="(part, index) in template.parts" :key="index"
          :style="{'background-color':part.color}"
        >
          <div class="part_content">
            {{ index + 1 }}.
            <input
              v-model="part.title"
              type="text"
              placeholder="part title"
            > 
            <div class="divider"></div>
            <label>
              <input type="checkbox" v-model="part.repeat"/>
              Repeat
            </label>
            <span v-if="part.repeat">
              &nbsp;<input
                v-model="part.timesRepeat"
                type="number"
                min="1"
                @blur="checkIfRepeatsFits(index)"
                > times
            </span>
            <div class="divider"></div>
            <div>
              Duration:
              <br>
              <input
                v-model="part.duration.hours"
                type="number"
                max="23"
                @blur="checkIfTimeFits(index, 'hours')"
                min="0"
                > h
              <input
                v-model="part.duration.minutes"
                type="number"
                max="59"
                @blur="checkIfTimeFits(index, 'minutes')"
                min="0"
                > m
              <input
                v-model="part.duration.seconds"
                type="number"
                max="59"
                @blur="checkIfTimeFits(index, 'seconds')"
                min="0"
                > s
            </div>
            <div class="divider"></div>
            <label> 
              <input type="checkbox" v-model="part.pauseOnFinish"/>
              Pause after finished</label>
            <br>
            <div class="divider"></div>
            <label> 
              <input type="checkbox" v-model="part.playAlarm"/>
              Play alarm</label>
            <br>
            <div class="divider"></div>
            Change color:
            <input 
              type="color" 
              v-model="part.color">
            <i 
              @click="removePart(index)"
              class="fa fa-trash fa-lg remove_part" 
              aria-hidden="true"
            ></i>
          </div>
        </div>
        <div 
          class="part add_part_container"
          @click="addPart"
        >
          <div class="add_part">
            <i class="fa fa-plus" aria-hidden="true"></i>
          </div>
        </div>
      </div>
    </div>
    <div v-if="curTab == 'templates'" id="content_templates">
      <div id="save_button">
        <div @click="templates.unshift(JSON.parse(JSON.stringify(template)))">
          Save current template
        </div>
      </div>
      <div
        v-for="(tl, index) in templates" :key="index"
      >
        <div
          class="template"
        >
          <div class="tl_index">
            {{ index+1 }}
          </div>
          <div class="tl_main">
            <div class="tl_control">
              <div
                class="tl_load"
                @click="template = JSON.parse(JSON.stringify(tl)); 
                curTab = 'editor'"
              >Load</div>
              <i 
                @click="removeTemplate(index)"
                class="fa fa-trash fa-lg" 
                aria-hidden="true"
              ></i>
            </div>
            <div class="tl_display">
              <div
                v-for="(part, index) in tl.parts" :key="index"
                class="tl_part"
                :style="{'background-color':part.color}"
              >
                <div class="part_content">
                  <div class="icons">
                    <i 
                      class="fa fa-step-forward" aria-hidden="true"
                      :class="{active: part.pauseOnFinish}"
                    ></i>
                    <i 
                      class="fa fa-refresh" aria-hidden="true"
                      :class="{active: part.repeat}"
                    ></i>
                    <i 
                    class="fa fa-volume-up" aria-hidden="true"
                    :class="{active: part.playAlarm}"
                  ></i>
                  </div>
                  <div class="time">
                    {{getTimeStringFromPart(part)}}
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="template_title">
          <div>{{ tl.title }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'App',
  data() { return {
    curTab: localStorage.getItem('curTab'),
    template: JSON.parse(localStorage.getItem('lastTemplate')),
    time: {},
    tabs: ['templates', 'editor', 'play'],
    templates: JSON.parse(localStorage.getItem('templates'))
  } },
  watch: {
    editMode() {
      if (this.editMode) {
        clearInterval(this.time.interval);
        clearInterval(this.time.templateIterval);
      } else {
        this.compileTemplate();
      }
    },
    curTab() {
      if (this.curTab != 'editor' && !this.editMode) {
        clearInterval(this.time.interval);
        clearInterval(this.time.templateIterval); 
      } else if (this.curTab == 'editor'
        && !this.editMode) {
        this.compileTemplate();
      }
    }
  },
  methods: {
    addPart() {
      let newPart = {
        title: '', 
        pauseOnFinish: false, 
        duration: {hours: 0, minutes: 0, seconds: 10}, 
        repeat: false, 
        color: getRandomHexColor(), 
        timesRepeat: 3,
        playAlarm: false
      };
      this.template.parts.push(newPart);
    },
    removePart(index) {
      this.template.parts.splice(index, 1)
    },
    removeTemplate(index) {
      this.templates.splice(index, 1)
    },
    checkIfTimeFits(index, value) {
      let maxTimeValues = {'seconds': 59, 'minutes': 59, 'hours': 23};
      let timeValueType = maxTimeValues[value];

      let curValue = this.template.parts[index].duration[value];
      this.template.parts[index].duration[value] = curValue > timeValueType ? timeValueType : curValue;
      curValue = this.template.parts[index].duration[value];
      this.template.parts[index].duration[value] = curValue < 0 ? 0 : curValue;

      let timeSum = 0;
      for (let timeKey of Object.keys(maxTimeValues)) {
        timeSum += this.template.parts[index].duration[timeKey];
      }
      if (timeSum == 0) {
        this.template.parts[index].duration['seconds'] = 1;
      }
    },
    checkIfRepeatsFits(index) {
      let curTimeRepeats = this.template.parts[index].timesRepeat;
      this.template.parts[index].timesRepeat = curTimeRepeats < 1 ? 1 : curTimeRepeats;
    },
    compileTemplate() {
      this.time = {};
      this.time.parts = [];
      this.time.curIndex = 0;
      this.time.loopIsRunning = false;
      this.time.startedLoop = false;
      this.time.spentTimeBeforePause = 0;
      this.time.widthPercentage = 0;
      for (let key in this.template.parts) {
        let part = this.template.parts[key];
        this.time.parts[key] = {
          widthPercentage: 0,
          loopsToDo: part.repeat ? part.timesRepeat : 1,
          loopsDone: 0,
          isLastPart: this.template.parts.length == Number(key) + 1,
          timeLeft: getTimeStringFromPart(part)
        }
      }
    },
    toogleLoop() {
      // starting the loop initially
      if (!this.time.startedLoop) {
        this.time.startedLoop = true;
        this.time.loopIsRunning = true;
        this.time.curIndex = 0;
        this.nextInterval();
        this.updateTemplateProgress();
      } else {
        // if went to other part then stop the loop
        if (this.time.shouldStop) {
          this.time.shouldStop = false;
          this.time.spentTimeBeforePause = 0;
          this.time.loopIsRunning = false;
          clearInterval(this.time.interval);          
          return;
        }
        // if loop is started then it will be paused or continued
        if (this.time.loopIsRunning) {
          // if is running then stop
          this.time.spentTimeBeforePause = 0;
          this.time.loopIsRunning = false;
          clearInterval(this.time.interval);
          this.time.spentTimeBeforePause = this.time.spentTime;
        } else {
          // continue the loop
          this.time.loopIsRunning = true;
          this.nextInterval();
        }
      }
    },
    assignDatesForPart(index) {
      this.time.paused = false;
      let date = new Date();
      let partDuration = this.template.parts[index].duration;
      this.time.endDate = new Date(date.setHours(
        date.getHours() + partDuration.hours,
        date.getMinutes() + partDuration.minutes,
        date.getSeconds() + partDuration.seconds
      ))
      this.time.startDate = new Date();
      this.time.fullDistance = 
        this.time.endDate.getTime() - 
        this.time.startDate.getTime();
    },
    nextInterval() {
      this.assignDatesForPart(this.time.curIndex);
      this.time.interval = setInterval(() => {
        // updating time 
        let index = this.time.curIndex;
        let now = new Date();
        this.time.spentTime = now.getTime() - this.time.startDate.getTime() + this.time.spentTimeBeforePause;
        let percentage = Math.floor(this.time.spentTime / this.time.fullDistance * 100);
        
        // check if part has finished
        if (percentage >= 100) {
          // clear current interval
          clearInterval(this.time.interval);
          this.time.spentTimeBeforePause = 0;
          let tPart = this.template.parts[index];
          let part = this.time.parts[index];
          part.widthPercentage = '100%';
          part.loopsDone += 1;
          // check if part repeated needed times
          if (part.loopsDone == part.loopsToDo) {
            part.loopsDone == 0;
            // play sound signal
            if (tPart.playAlarm) {
              new Audio(require('./assets/sound/alarm1.mp3')).play()
            }
            // check if it's last part
            if (part.isLastPart) {
              // if it's last part go to next loop
              clearInterval(this.time.templateIterval);
              this.compileTemplate();
              // if it pauses on finish then we do not start next loop
              if (!tPart.pauseOnFinish) {
                this.toogleLoop();
              }
            } else {
              // otherwise go to next part
              this.time.spentTime = 0;
              this.time.curIndex += 1;
              part.timeLeft = '-';
              // if part should stop on finish then we do not go to next interval
              if (!tPart.pauseOnFinish) {
                this.nextInterval();
              } else {
                this.time.loopIsRunning = false;
              }
            }
          } else {
            // if part hasn't went through all loops - do one more
            this.time.spentTime = 0;
            this.nextInterval();
          }
        } else {
          // update current interval
          this.time.parts[index].widthPercentage = percentage + '%';
          this.time.parts[index].timeLeft = getTimeFromMilliseconds(this.time.fullDistance - this.time.spentTime);
        }
      }, 200);
    },
    handlePartClick(index) {
      let curIndex = this.time.curIndex;
      // check whether we clicked on part which is running now
      if (curIndex == index) {
        // stop or resume the loop
        this.toogleLoop();
      } else if (curIndex - 1 == index && this.template.parts[index].pauseOnFinish && this.time.loopIsRunning == false) {
        // case when previous part was pause-on-finish
        console.log('do');
        this.toogleLoop();
      } else {
        if (!this.time.startedLoop) {
          this.toogleLoop();
          this.handlePartClick(index);
        }
        // loop through all part and set some props
        for (let key in this.time.parts) {
          let part = this.time.parts[key];
          // check part's index
          if (key < index) {
            // if part is before current then we make finished
            part.loopsDone = part.loopsToDo;
            part.widthPercentage = '100%';
            part.timeLeft = '-';
          } else {
            // or reset the part
            part.loopsDone = 0;
            part.widthPercentage = '0%';
            part.timeLeft = getTimeStringFromPart(this.template.parts[key]);
          }
        }
        // then set current index and continue loop
        this.time.shouldStop = true;
        this.toogleLoop();
        this.time.curIndex = index;
        this.toogleLoop(); 
      }
    },
    updateTemplateProgress() {
      let templateDuration = getTemplateDuration(this.template);
      this.time.updateIntervalTime = templateDuration / 100;
      this.time.templateIterval = setInterval(() => {
        // check if loops isn't paused
        if (this.time.loopIsRunning) {
          let timeSpent = getTemplateTimeSpent(this.template, this.time);
          // console.log(time);
          this.time.widthPercentage = (timeSpent / templateDuration * 100) + '%';
        }
      }, 200)
    },
    saveSomeData() {
      localStorage.setItem('lastTemplate', JSON.stringify(this.template));
      localStorage.setItem('templates', JSON.stringify(this.templates));
      localStorage.setItem('curTab', this.curTab);
      // localStorage.setItem('editMode', JSON.stringify(this.editMode));
    },
    getTimeStringFromPart(part) {
      return getTimeStringFromPart(part);
    }
  },
  beforeCreate() {
    // check whether user is first time on this web site
    if (!localStorage.getItem('firstTimeHere')) {
      localStorage.setItem('firstTimeHere', 'true');
      localStorage.setItem('curTab', 'play');
      localStorage.setItem('lastTemplate', JSON.stringify(getExampleTemplate()));
      localStorage.setItem('templates', JSON.stringify(getFirstTemplates()));
    }
  },
  beforeMount() {
    this.compileTemplate();
  },
  mounted() {
    // save some data
    window.onbeforeunload = this.saveSomeData;
  }
}

function getTemplateDuration (template) {
  let parts = template.parts;
  let fullDuration = 0;
  for (let part of parts) {
    let partDuration = getPartDuration(part);
    let timesRepeat = 1;
    if (part.repeat) {
      timesRepeat = part.timesRepeat
    }
    for (let i = 0; i < timesRepeat; i++) {
      fullDuration += partDuration;
    }
  }
  return fullDuration;
  function getPartDuration (part) {
    let duration = part.duration.hours * 3600
      + part.duration.minutes * 60
      + part.duration.seconds;
    return duration;
  }
}
function getTemplateTimeSpent (template, time) {
  let parts = time.parts;
  let timeSpent = 0;
  for (let key in time.parts) {
    let part = time.parts[key];
    let templatePart = template.parts[key];
    // check if part is finished
    if (part.widthPercentage == '100%' 
      && (!templatePart.repeat || part.loopsDone == part.loopsToDo)) {
      let partDuration = getPartDuration(templatePart);
      // console.log('partDuration: ' + partDuration);
      let timesRepeat = 1;
      if (templatePart.repeat) {
        timesRepeat = templatePart.timesRepeat
      }
      for (let i = 0; i < timesRepeat; i++) {
        timeSpent += partDuration;
      }
    }
  }

  if (template.parts[time.curIndex].repeat) {
    let part = time.parts[time.curIndex];
    let partDuration = getPartDuration(template.parts[time.curIndex]);
    // console.log(part.loopsDone);
    for (let i = 0; i < part.loopsDone; i++) {
      timeSpent += partDuration;
    }
  }
  // console.log('before: ' + timeSpent);
  // console.log('spentTime: ' + time.spentTime);
  timeSpent += time.spentTime / 1000;
  // console.log('after: ' + timeSpent);
  return timeSpent;

  function getPartDuration (part) {
    let duration = part.duration.hours * 3600
      + part.duration.minutes * 60
      + part.duration.seconds;
    return duration;
  }
}
function getTimeStringFromPart (part) {
  let resultString = '';
  let dur = part.duration;
  let timeNames = [
    {key: 'hours', short: 'h'},
    {key: 'minutes', short: 'm'},
    {key: 'seconds', short: 's'}
  ];
  for (let key in timeNames) {
    let timeName = timeNames[key];
    if (dur[timeName.key] != 0) {
      resultString += String(dur[timeName.key]) + timeName.short;
      resultString += ' ';
    }  
  }
  resultString = resultString.trim();
  return resultString == '' ? '-' : resultString;
}
function getTimeFromMilliseconds (mills) {
  let resultString = '';
  let secs = Math.floor(mills / 1000) + 1;
  let timeValues = [
    {short: 'h', timeValue: Math.floor(secs / 60 / 60)},
    {short: 'm', timeValue: Math.floor((secs / 60) % 60)},
    {short: 's', timeValue: Math.floor(secs % 60)}
  ];
  for (let key in timeValues) {
    let value = timeValues[key];
    if (value.timeValue != 0) {
      resultString += String(value.timeValue) + value.short;
      resultString += ' ';
    }  
  }
  resultString = resultString.trim();
  return resultString == '' ? '-' : resultString;
}
function getRandomHexColor () {
  let h = Math.floor(Math.random() * 360);
  let s = 70;
  let l = 70;

  l /= 100;
  const a = s * Math.min(l, 1 - l) / 100;
  const f = n => {
    const k = (n + h / 30) % 12;
    const color = l - a * Math.max(Math.min(k - 3, 9 - k, 1), -1);
    return Math.round(255 * color).toString(16).padStart(2, '0');   // convert to Hex and prefix "0" if needed
  };
  return `#${f(0)}${f(8)}${f(4)}`;
}
function getFirstTemplates () {
  return [{
    title: 'Pomodoro',
    looped: true,
    parts: [
      {title: 'work', pauseOnFinish: false, duration: {hours: 0, minutes: 40, seconds: 0}, repeat: false, color: getRandomHexColor(), timesRepeat: 3, playAlarm: false},
      {title: 'rest', pauseOnFinish: false, duration: {hours: 0, minutes: 20, seconds: 0}, repeat: false, color: getRandomHexColor(), timesRepeat: 3, playAlarm: false}
    ]
  },
  {
    title: 'Simple countdown',
    looped: true,
    parts: [
      {title: '', pauseOnFinish: false, duration: {hours: 0, minutes: 1, seconds: 0}, repeat: false, color: getRandomHexColor(), timesRepeat: 3, playAlarm: false}
    ]
  }];
}
function getExampleTemplate () {
  return {
    title: 'Example loop',
    looped: true,
    parts: [
      {title: 'simple part', pauseOnFinish: false, duration: {hours: 0, minutes: 0, seconds: 5}, repeat: false, color: getRandomHexColor(), timesRepeat: 3},
      {title: 'repeats', pauseOnFinish: false, duration: {hours: 0, minutes: 0, seconds: 3}, repeat: true, color: getRandomHexColor(), timesRepeat: 3},
      {title: 'has alarm', pauseOnFinish: true, duration: {hours: 0, minutes: 0, seconds: 5}, repeat: false, color: getRandomHexColor(), timesRepeat: 3, playAlarm: 'true'},
      {title: 'pauses on finish', pauseOnFinish: true, duration: {hours: 0, minutes: 0, seconds: 5}, repeat: false, color: getRandomHexColor(), timesRepeat: 3},
      {title: 'last part', pauseOnFinish: false, duration: {hours: 0, minutes: 0, seconds: 20}, repeat: false, color: getRandomHexColor(), timesRepeat: 3}
    ],
  };
}
</script>

<style lang="sass">

</style>
