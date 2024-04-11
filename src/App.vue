<template>
  <v-app>
    <v-main>
      <v-container fluid>
        <v-card class = "card">
          
          <v-toolbar color = "primary">
            
            <v-toolbar-title class = "text-center"> 
              <v-icon icon = "$menu"></v-icon>FRAMEWORKS
            </v-toolbar-title>
            
            <v-dialog v-model = "dialog" max-width="500" transition="dialog-bottom-transition">
              <template v-slot:activator="{ props }">
                <v-btn
                v-bind="props" 
                class = "add" 
                elevation = 4 
                prepend-icon = "mdi-plus-circle" @click = "updateDialog = false;
                resetAddForm();">
                Add
              </v-btn>
            </template>
            
            <template v-slot:default="{ isActive }">
              <v-form @submit.prevent="addItem" validate-on="submit">
              <v-card>
                <v-toolbar color = "primary"  v-if="updateDialog == false">
                  
                  <v-toolbar-title> 
                    <v-icon icon = "mdi-plus-circle"></v-icon>Add Task
                  </v-toolbar-title>
                </v-toolbar>

                <v-toolbar color = "primary"  v-if="updateDialog == true">
                  
                  <v-toolbar-title> 
                    <v-icon icon = "mdi-square-edit-outline"></v-icon>Edit Task
                  </v-toolbar-title>
                </v-toolbar>
                
                  <v-card-text>
                    <v-text-field label = "Title"  
                      id="validate-title"
                      v-model="itemTitle"
                      variant="outlined"
                      :rules="titleRule"
                      v-if="updateDialog == false"></v-text-field>
                    <v-text-field label = "Description"
                      id="validate-description"
                      v-model="itemDescription"
                      variant="outlined"
                      :rules="descriptionRule"></v-text-field>
                    <v-menu
                      v-model="menu2"
                      :close-on-content-click="false"
                      :nudge-right="40"
                      lazy
                      transition="scale-transition"
                      offset-y
                      full-width
                    >
                      <template v-slot:activator="{ on }">
                        <v-text-field
                          type="text"
                          v-model="reformattedDate"
                          label="Deadline"
                          append-inner-icon="mdi-calendar"
                          readonly
                          variant="outlined"
                          v-on="on"
                          @click="menu2 = !menu2"
                        ></v-text-field>
                      </template>
                      <v-date-picker
                        v-model="itemDate"
                        no-title
                        color="primary"
                        @input="reformatDate"
                        @change="menu2 = false"
                      ></v-date-picker>
                    </v-menu>
                    <v-container fluid>
                      <v-radio-group 
                      v-model="itemPriority" 
                      label = "Priority" 
                      off-icon="mdi-radiobox-blank"
                      on-icon="mdi-radiobox-marked" inline>
                      <v-radio label="Low" value="Low"></v-radio>
                      <v-radio label="Medium" value="Medium"></v-radio>
                      <v-radio label="High" value="High"></v-radio>
                      </v-radio-group>
                    </v-container>
                   
                  
                </v-card-text>
              
              
              
              <v-card-actions class = "justify-right">
                <v-spacer></v-spacer>
                <v-btn
                  v-if="updateDialog == false"
                  type = "submit"
                  prepend-icon = "mdi-plus-circle"
                  style="width: 100px"
                  text = "ADD"
                  @click ="addItem"
                  elevation = 2
                  class = "submit"
                  
                >
                
                
                </v-btn>
                <v-btn
                  v-if="updateDialog == true"
                  class = "submit"
                  @click="editItem"
                  type="submit"
                  style="width: 100px"
                  prepend-icon = "mdi-square-edit-outline"
                  text = "Edit"
                >
                </v-btn>
                <v-btn
                class = "cancel"
                prepend-icon = "mdi-cancel"
                text="CANCEL"
                @click="dialog = false"
                elevation=2
                ></v-btn>
              
              </v-card-actions>
           
            </v-card>
          </v-form>
          </template>
        </v-dialog>
        
      </v-toolbar>
      
      <v-card-text>
        <v-table>
          
          <thead>
            <tr>
              <th class="text-center">Title</th>
              <th class="text-center">Description</th>
              <th class="text-center">Deadline</th>
              <th class="text-center">Priority</th>
              <th class="text-center">Is Complete</th>
              <th class="text-center">Action</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in items" :key="item.Title">
              <td class="text-center">{{ item.Title }}</td>
              <td class="text-center">{{ item.Description }}</td>
              <td class="text-center">{{ item.Deadline }}</td>
              <td class="text-center">{{ item.Priority }}</td>
              <td class="text-center">
                <v-checkbox
                  v-model="item.isComplete"
                  @click="removeUpdateButton()"
                  style="
                    display: flex;
                    align-items: center;
                    justify-content: center;
                  "
                ></v-checkbox>
              </td>
              <td>
                <v-btn
                    color="primary"
                    size="small"
                    @click="
                      updateDialog = true;
                      dialog = true;
                      populateUpdateItem(item);
                    "
                    v-if="!item.isComplete"
                    v-bind= "props"
                  >
                  <v-icon icon = "mdi-square-edit-outline"></v-icon>
                  UPDATE
                </v-btn>
                <v-btn
                color="red"
                size="small"
                @click="
                deleteItem(index);">
                  <v-icon icon = "$cancel"></v-icon>
                  DELETE
                </v-btn>
              </td>
              
            </tr>
          </tbody>
          
        </v-table>
      </v-card-text>
    </v-card>
  </v-container>
</v-main>



</v-app>
</template>


<script>
import moment from 'moment';
import toastr from 'toastr';

export default {
  components: {
    
  },
  data() {
    return {
      menu2: false,
      dialog: false,
      updateDialog: false, 
      itemPriority: 'Low', // Initialize the variable with a default
      itemTitle: '',
      itemDescription: '',
      itemDate: null,
      reformattedDate: moment().format('MM/DD/YYYY'),

      titleRule: [
        (value) => {
          if (value) {
            // Check for uniqueness
            const isUnique = !this.items.some((item) => item.Title === value);
            // Return true if the title is unique, otherwise return an error message
            return isUnique || 'Title must be unique';
          }
          return 'Title is Required!';
        },
      ],
      descriptionRule: [
        (value) => {
          if (value) return true;
          return 'Description is Required!';
        },
      ],

      items: [],
    };
  },
  methods: {
    initDate() {
      this.reformattedDate = moment().format('MM/DD/YYYY');
    },

    reformatDate() {
      this.reformattedDate = moment(this.itemDate).format('MM/DD/YYYY');
    },

    resetAddForm() {
      // Reset input values
      this.itemTitle = '';
      this.itemDescription = '';
      this.itemPriority = 'low';
      this.itemDate = null;
      this.reformattedDate = moment().format('MM/DD/YYYY');
    },

    addItem() {
      // validate for empty
      if (this.itemTitle == '' || this.itemDescription == '') return;

      // validate for uniqueness
      const isTitleValid = this.titleRule[0](this.itemTitle);
      if (isTitleValid !== true) return;

      // if all validation passes then we can add the new entry
      const newItem = {
        Title: this.itemTitle,
        Description: this.itemDescription,
        Deadline: this.reformattedDate,
        Priority: this.itemPriority,
        isComplete: false,
      };

      //Add new entry to table and close dialog
      this.items.push(newItem);
      this.dialog = false;

      // Reset input values
      this.itemTitle = '';
      this.itemDescription = '';
      this.itemPriority = 'low';
      this.itemDate = null;

      // display toaster
      toastr.options.positionClass = 'toast-bottom-right';
      toastr.success('Task was Successfully Added');
    },

    deleteItem(index) {
      this.items.splice(index, 1);
      toastr.options.positionClass = 'toast-bottom-right';
      toastr.success('Task was successfully deleted');
    },

    populateUpdateItem(item) {
      // Set the values for the dialog based on the selected item
      this.itemTitle = item.Title;
      this.itemDescription = item.Description;
      this.reformattedDate = item.Deadline;
      this.itemPriority = item.Priority;
    },

    editItem() {
      //find the index of the given item
      
      const index = this.items.findIndex(
        (item) => item.Title === this.itemTitle
      );

      // Update the item with the new values
      this.items[index].Description = this.itemDescription;
      this.items[index].Deadline = this.reformattedDate;
      this.items[index].Priority = this.itemPriority;

      // Reset the form and close the dialog
      this.resetAddForm();
      this.dialog = false;

      // Display toaster
      toastr.options.positionClass = 'toast-bottom-right';
      toastr.success('Task was successfully updated');
    },
  },
  watch: {
    // Watch for changes in itemDate and format it when it changes
    itemDate: {
      handler: 'reformatDate',
      immediate: true, // Format initially as well
    },
  },
};
</script>

<style>
.add{
  background-color: #1976D2 !important;
}
.cancel{
  color: white !important;
  background-color: red !important;
}
.submit{
  color: white !important;
  background-color: #1976D2 !important;

}
button{
  padding: 0.6em 1.2em;
}
</style>
