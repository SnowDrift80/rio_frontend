<style scoped>
  /* import scoped css file */
  @import '@/assets/css/converse.css';
</style>

<template>
  <v-container class="full-page-container text-center">
    <v-row class="headliner d-flex justify-center">
      <v-col cols="12" md="2">
        <div>
          <v-img max-height="60" src="@/assets/triz_participacoes.webp"></v-img>
        </div>
      </v-col>
      <v-col cols="12" md="10">
        <div class="headline-wrapper">
          <div id="cursor" style="width: 10px; height: 20px; background-color: transparent;"></div>
        <!--  <div id="cursor" style="display: inline-block; width: 10px; height: 20px; background-color: transparent;"></div> -->
        </div>
      </v-col>
    </v-row>
  <!--  <v-row class="first-row content-wrapper" style="align-items: center;" justify-center> -->
    <v-row class="first-row content-wrapper" style="align-items: center;" justify-center>
      <v-row class="first-row  protocol-wrapper" style="height: 92%;">
        <div class="px-6" style="width: 96%; margin: 0 auto; padding: 16px; overflow-y: auto" id="protocolview"></div>
      </v-row>
    </v-row>
    <v-row class="spacer"></v-row>
    <v-row class="second-row content-wrapper" justify-center>
      <v-col cols="12" md="10">
        <div>
          <v-form @submit.prevent="submitForm">
              <v-textarea 
                v-model="message" 
                label="Sua pergunta" 
                required 
                no-resize
                rows="3"
                hide-details
                @keydown.ctrl.enter="submitForm"
                ></v-textarea>
          </v-form>
        </div>
      </v-col>
      <v-col cols="12" md="2">
        <div align-center>
          <v-btn block color="green" @click="submitForm">enviar</v-btn>
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>


<script>
export default {
  data() {
    return {
      message: '',
      messages: [], // array to store user submitted messages
      sessionId: ''
    }
  },
  methods: {
    submitForm() {
      // Get the message content and split it by line breaks
      const messageContent = this.message.trim().split('\n')
      
      console.log("sessionId:", this.sessionId)

      // clear v-textarea
      this.message = '';

      // add the message to the messages array
      this.messages.push(...messageContent);


      // create the json object containing message and session_id
      const data = {
        message: messageContent.join('\n'),
        session_id: this.sessionId || '' // send empty string if this.sessionId not set
      };

      console.log("json data: ", data)

      // start blinking cursor
      const blinkInterval = setInterval(toggleCursor, 250); // adjust blink interval here

      // append each line to the chat protocol
      const protocolView = document.getElementById('protocolview');
      messageContent.forEach((line, index) => {
        const paragraph = document.createElement('p');
        paragraph.textContent = line;
        protocolView.appendChild(paragraph);

        // Add an empty line after each post, except for the last one
        if (index == messageContent.length - 1) {
          protocolView.appendChild(document.createElement('br'));
        }
      });

        // Send the form data to your server
//      fetch('http://192.168.1.110:5000/api/converse', {
      fetch('http://89.116.73.94:5000/api/converse', { 
        method: 'POST',
        headers: {
          'Content-Type': 'application/json' // headers to indicate json payload
        },
        body: JSON.stringify(data) // convert data to json
      })
      .then(response => response.json()) // parse response as JSON
      .then(data => {
        // stop blinking wehn response is received
        //data.text().then(data => {
          clearInterval(blinkInterval); // stop blinking 
          // Handle the response from the server
          setCursorTransparent();
          console.log('Response from server:', data);

          // extract response message and session id from json
          const { message, session_id } = data;

          // update sessionId
          this.sessionId = session_id;

          // Process the response data
          const messageContent = message.split('\n');
          this.messages.push(...messageContent);
          messageContent.forEach((line, index) => {
            const paragraph = document.createElement('p');
//            paragraph.className = 'text-justify px-6 py-2';
            paragraph.className = 'px-6 py-2';
            paragraph.innerHTML = line;
            paragraph.style.backgroundColor = '#e3ffea';
            protocolView.appendChild(paragraph);

            // Add an empty line after each post, except for the last one
            if (index == messageContent.length - 1) {
              protocolView.appendChild(document.createElement('br'));
            }
          });
          // scroll to the bottom of the protocol view
          // const lastParagraph = protocolView.querySelector('p:last-child');
          const paragraphs = protocolView.querySelectorAll('p');
          const lastParagraph = paragraphs[paragraphs.length - 1];

          // Scroll the last paragraph into view
          lastParagraph.scrollIntoView({block: 'end' });
          console.log(protocolView)
          console.log(lastParagraph)
        // });
      })
      .catch(error => {
        // Handle any errors that occur during the request
        console.error('Error:', error);
      });

      // Function to toggle cursor visibility
      function toggleCursor() {
        console.log("CURSOR BLINK")
        const cursor = document.getElementById('cursor');
        cursor.style.backgroundColor = cursor.style.backgroundColor === 'red' ? 'transparent' : 'red';
      }
      function setCursorTransparent() {
        console.log("CURSOR TRANSPARENT")
        const cursor = document.getElementById('cursor');
        cursor.style.backgroundColor = 'transparent';
      }
    }
  }
}
</script>

