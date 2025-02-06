# Tkinter-Data-App

## Notes from programming Data App

Below is the app I created to process csv file data for call log sheets 

**Important**: No data will be included just the source python file used to create the app.

### Points of Interest: 

- Tkinter GUI 
- reading / writing files 
- functional-ish based prgramming to handle processing

#### Input data headers (for reference)
``` python
Id,Initiator Device,Initiator Type,Provider Name,Room,Patient Name,Room Count,Call Start,Call End,Call Origin,Status,Call Duration,Call Type,Used Audio,Used Video,Used Screen,Email,Patient MRN,Patient CSN,Care Event Type,Invited Participants,Stat Alarm Activated
```

## Main source code file: 

``` python
from tkinter import *
from tkinter import filedialog
from tkinter import ttk
import csv
import time
import os

class App(Tk):
    def __init__(self):
        super().__init__()

        # Title, window size
        self.title("TV Kit Data Ingest")
        self.geometry('550x400')
        self.resizable(width=False, height=False)

        # Colors
        self.color1 = '#4f4f4f'
        self.color2 = '#6c98e0'
        self.color3 = '#385380'
        self.color4 = 'WHITE'

        # Creating Tabs (Notebook and Frames)
        self.my_notebook = ttk.Notebook(self)
        self.my_notebook.pack(fill='both', expand=1)

        self.export_frame = Frame(self.my_notebook, width=550, height=400)
        self.use_case_frame = Frame(self.my_notebook, width=550, height=400)
        self.specialties_frame = Frame(self.my_notebook, width=550, height=400)

        self.export_frame.pack(fill='both', expand=1)
        self.use_case_frame.pack(fill='both', expand=1)
        self.specialties_frame.pack(fill='both', expand=1)

        # Adds Frame to Notebook 
        self.my_notebook.add(self.export_frame, text="Export")
        self.my_notebook.add(self.use_case_frame, text="Edit Use Cases List")
        self.my_notebook.add(self.specialties_frame, text="Edit Specialties List")

        # Create Widgets for Export Tab
        # File-in Widgets
        self.label_in = Label(self.export_frame, text='Open File Location:')
        self.label_in.grid(row=1, column=0, padx=5, pady=12)

        self.text_box_in = Text(self.export_frame, width=40, height=1)
        self.text_box_in.grid(row=1, column=1, padx=5, pady=12)

        self.file_in_button = Button(self.export_frame, text="Select File", command=self.file_in)
        self.file_in_button.grid(row=1, column=2, padx=5, pady=12)

        self.export_title = Label(self.export_frame, text='Ingest Call Log Data', font=("Arial", 14, "bold"))
        self.export_title.grid(row=0, column=1, padx=5, pady=12)

        # File-out Widgets
        self.label_out = Label(self.export_frame, text='Output File Location:')
        self.label_out.grid(row=2, column=0, padx=5, pady=12)

        self.text_box_out = Text(self.export_frame, width=40, height=1)
        self.text_box_out.grid(row=2, column=1, padx=5, pady=12)

        self.file_out_button = Button(self.export_frame, text="Save As", command=self.file_out)
        self.file_out_button.grid(row=2, column=2, padx=5, pady=12)

        # Ingest Data Button
        self.submit_button = Button(self.export_frame,
                                    background=self.color2, 
                                    foreground=self.color4,
                                    activebackground=self.color3,
                                    activeforeground=self.color4,
                                    highlightthickness=2,
                                    highlightbackground=self.color2,
                                    highlightcolor='WHITE',
                                    text="Ingest Data",
                                    cursor='hand2',
                                    font=('Arial', 10, 'bold'),
                                    command=self.submit)
        self.submit_button.grid(row=3, column=1, padx=5, pady=12)
        
        self.progress_bar = ttk.Progressbar(self.export_frame, orient=HORIZONTAL, length=300, mode='determinate')

        self.file_in_error = Label(self.export_frame, text="Please Provide a File In Path!", fg="RED")
        self.file_out_error = Label(self.export_frame, text="Please Provide a File Out Path!", fg="RED")


        # Use Case Pane
        self.use_case_button = Button(self.use_case_frame,
                                    background=self.color2, 
                                    foreground=self.color4,
                                    activebackground=self.color3,
                                    activeforeground=self.color4,
                                    highlightthickness=2,
                                    highlightbackground=self.color2,
                                    highlightcolor='WHITE',
                                    text="Open Use Cases List",
                                    cursor='hand2',
                                    font=('Arial', 10, 'bold'),
                                    command=self.open_csv_dict_use_cases)
        self.use_case_button.grid(row=3, column=1, padx=5, pady=2)

        self.use_case_sort = Button(self.use_case_frame, text="Sort List", command=self.sort_csv_dict_use_cases)
        self.use_case_sort.grid(row=2, column=0, padx=5, pady=12)

        self.use_case_instructions = Button(self.use_case_frame, text="Open Instructions", command=self.open_use_case_instructions)
        self.use_case_instructions.grid(row=2, column=2, padx=5, pady=12)

        self.use_case_title = Label(self.use_case_frame, text='Use Case Category Menu', font=("Arial", 14, "bold"))
        self.use_case_title.grid(row=0, column=1, padx=5, pady=6)

        self.use_case_body_text = Label(self.use_case_frame, text=" Edit the list of callers and thier associated use cases \n by clicking on the 'Open Use Cases List' button below. \n This will open an excel sheet and allow you to add or \n update to the list of use cases. \n \n For further instruction click on 'Open Instructions' \n \n To sort this excel list press 'Sort List' button\n ", justify='left')
        self.use_case_body_text.grid(row=1, column=1, padx=5, pady=6)

        self.use_case_important_note = Label(self.use_case_frame, text=" Important Note: \n - PLEASE SAVE & CLOSE the excel window before returning to \n the application. Otherwise it way result in errors in the output.", justify='left')
        self.use_case_important_note.grid(row=4, column=1, padx=5, pady=12)

        self.error_finding_use_case = Label(self.use_case_frame, text="Unable to find 'use_case_key_values.csv' in folder", fg="RED")



        # Specialties Pane
        self.specialties_button = Button(self.specialties_frame,
                                            background=self.color2, 
                                            foreground=self.color4,
                                            activebackground=self.color3,
                                            activeforeground=self.color4,
                                            highlightthickness=2,
                                            highlightbackground=self.color2,
                                            highlightcolor='WHITE',
                                            text="Open Specialties List",
                                            cursor='hand2',
                                            font=('Arial', 10, 'bold'),
                                            command=self.open_specialties)
        self.specialties_button.grid(row=3, column=1, padx=5, pady=12)

        self.specialties_sort = Button(self.specialties_frame, text="Sort List", command=self.sort_csv_dict_specialties)
        self.specialties_sort.grid(row=2, column=0, padx=5, pady=12)

        self.specialties_instructions = Button(self.specialties_frame, text="Open Instructions", command=self.open_specialties_instructions)
        self.specialties_instructions.grid(row=2, column=2, padx=5, pady=12)

        self.specialties_title = Label(self.specialties_frame, text='Provider Specialties Menu', font=("Arial", 14, "bold"))
        self.specialties_title.grid(row=0, column=1, padx=5, pady=6)

        self.specialties_body_text = Label(self.specialties_frame, text=" Edit the list of providers and thier associated specialties \n by clicking on the 'Open Specialties List' button below. \n This will open an excel sheet and allow you to add or \n update to the list of specialties. \n \n For further instruction click on 'Open Instructions' \n \n To sort this excel list press 'Sort List' button\n ", justify='left')
        self.specialties_body_text.grid(row=1, column=1, padx=5, pady=6)

        self.specialties_important_note = Label(self.specialties_frame, text=" Important Note: \n - PLEASE SAVE & CLOSE the excel window before returning to \n the application. Otherwise it way result in errors in the output.", justify='left')
        self.specialties_important_note.grid(row=4, column=1, padx=5, pady=12)

        self.error_finding_specialties = Label(self.specialties_frame, text="Unable to find 'specialties_key_values.csv' in folder", fg="RED")

    def file_in(self):
        self.text_box_in.delete(1.0, END)
        self.my_file_in = filedialog.askopenfilename(
            initialdir="",
            title="Select a File", 
            filetypes=(("csv files", "*.csv"), ("All Files", "*.*"))
            )
        
        if self.my_file_in:
            self.text_box_in.insert(END, self.my_file_in)
            self.file_in_error.grid_forget()



    def file_out(self):
        self.text_box_out.delete(1.0, END)
        self.my_file_out = filedialog.asksaveasfilename(
            initialdir="",
            title="Select Output File Locaiton",
            defaultextension=".csv", 
            filetypes=(("csv files", "*.csv"), ("All Files", "*.*"))
            )
        
        if self.my_file_out: 
            self.text_box_out.insert(END, self.my_file_out)
            self.file_out_error.grid_forget()

    
    def submit(self):
        self.progress_bar.grid(row=4, column=1, padx=5, pady=12)
        self.progress_bar['value'] = 0

        if not self.text_box_in.get(1.0, "end-1c"):
            self.file_in_error.grid(row=5, column=1, padx=5, pady=5)

        
        # All the processing
        working_table = get_useful_file_rows(self.my_file_in)
        processed_table = all_processing(working_table)

        if not self.text_box_out.get(1.0, "end-1c"):
            self.file_out_error.grid(row=5, column=1, padx=5, pady=5)

        output_csv(processed_table, self.my_file_out)

        for x in range(10):
            self.progress_bar['value'] += 10
            self.update_idletasks()
            time.sleep(0.03)

        os.startfile('"' + self.my_file_out + '"')


    def open_specialties(self):
        self.error_finding_specialties.grid_forget()
        self.specialties_important_note.grid(row=4, column=1, padx=5, pady=12)
        try:
            os.startfile('specialties_key_values.csv')
        except:
            self.error_finding_specialties.grid(row=4, column=1, padx=5, pady=5)
            self.specialties_important_note.grid(row=5, column=1, padx=5, pady=5)

    def sort_csv_dict_specialties(self):
        sort_dict_csv('specialties_key_values.csv')
    
    def sort_csv_dict_use_cases(self):
        sort_dict_csv('use_case_key_values.csv')

    def open_csv_dict_use_cases(self):
        self.error_finding_use_case.grid_forget()
        self.use_case_important_note.grid(row=4, column=1, padx=5, pady=12)
        try: 
            os.startfile('use_case_key_values.csv')
        except:
            self.error_finding_use_case.grid(row=4, column=1, padx=5, pady=5)
            self.use_case_important_note.grid(row=5, column=1, padx=5, pady=12)



    def open_use_case_instructions(self):
        os.startfile('use_case_instructions.txt')

    def open_specialties_instructions(self):
        os.startfile('specialties_instructions.txt')





def get_useful_file_rows(file_path_in):

    useful_table = []

    with open(file_path_in, 'r', newline='') as csvfile: 
        original_table = csv.reader(csvfile, delimiter=',', quotechar='"')
        
        id_index = 0
        provider_name_index = 3
        room_index = 4
        quantity_index = 6
        date_index = 7
        status_index = 10
        call_duration_index = 11

        list_table = list(original_table)
        header_row = list_table[0]

        for column in header_row:
            if column == 'Id':
                id_index = header_row.index(column)
            if column == 'Provider Name':
                provider_name_index = header_row.index(column)
            if column == 'Room':
                room_index = header_row.index(column)
            if column == 'Room Count':
                quantity_index = header_row.index(column)
            if column == 'Call Start':
                date_index = header_row.index(column)
            if column == 'Status':
                status_index = header_row.index(column)
            if column == 'Call Duration':
                call_duration_index = header_row.index(column)
                

        for row in list_table: 
            id = row[id_index]
            provider_name = row[provider_name_index]
            room = row[room_index]
            quantity = row[quantity_index]
            date = row[date_index]
            status = row[status_index]
            call_duration = row[call_duration_index]
            
            table_row = [id, provider_name, room, date, call_duration, status, quantity]
            useful_table.append(table_row)

            
        print(useful_table[0]) 

        useful_table[0][3] = 'Date'
        useful_table[0][6] = 'Quantity'

    return useful_table 


def output_csv(new_table, file_path_out):
    with open(file_path_out, 'w', newline='') as csvfile: 
        writer = csv.writer(csvfile)
        writer.writerows(new_table)


def all_processing(working_table): 
    updated_data_table = add_specialty(
        add_use_cases(
        add_locations(
        reformat_date(
        change_to_failed_status(
        reformat_duration(working_table)
        )))))


    return updated_data_table


def reformat_duration(table): 
    updated_table = table
    
    for row in updated_table: 
        time_values = row[4].split(':')

        total_in_minutes = 0
        if time_values[0] != 'Call Duration':
            seconds = int(time_values[2]) / 60
            minutes = int(time_values[1])
            hours = int(time_values[0]) * 60
            total_in_minutes = hours + minutes + seconds
            total_in_minutes = round(total_in_minutes, 3)
            row[4] = total_in_minutes

        if time_values[0] == 'Call Duration':
            row[4] = 'Call Duration (minutes)'
        
    return updated_table


def change_to_failed_status(table):
    status_table = table

    for row in status_table: 
        status = row[5]
        if status != 'Successful' and status != 'Status':
            row[5] = 'Failed'

    return status_table 


def reformat_date(table):
    date_table = table

    for row in date_table: 
        date_value_list = row[3].split(' ')
        if date_value_list[0][0] == '0':
            date_value_list[0] = date_value_list[0][1:]
        
        row[3] = date_value_list[0]

    return date_table

def add_locations(table): 
    location_table = table

    for row in location_table: 
        room = row[2]
        if 'OW' in room: 
            row.append('Geisinger Campus')
        elif 'CA' in room: 
            row.append('Carbon Campus')
        elif 'MI' in room: 
            row.append('Miners Campus')
        else: 
            row.append('Other')

    location_table[0][7] = 'Locations'
    return location_table

# handles dictionaries

def get_key_values_for_dict(filename):
    values_dict = {}
    with open(filename, newline='') as csvfile: 
        key_values = csv.reader(csvfile, delimiter=',', quotechar='|') 

        for row in key_values:
            key = row[0]
            value = row[1]
            values_dict[key] = value
            
    return values_dict 


def sort_dict_csv(filename):
    unsorted_dict = get_key_values_for_dict(filename)
    sorted_items = sorted(unsorted_dict.items())

    sorted_dict = dict(sorted_items)
    output_dict(sorted_dict, filename)


def output_dict(dict, filename):
    output_list = []
    
    for key in dict:
        value = dict[key]
        key_value_pair = [key, value]
        output_list.append(key_value_pair)
    
    with open(filename, 'w', newline='') as csvfile:
        writer = csv.writer(csvfile)
        writer.writerows(output_list)


def add_use_cases(table):
    use_case_table = table

    use_cases_dict = get_key_values_for_dict('use_case_key_values.csv')

    for row in use_case_table:
        provider = row[1]

        category = 'Misc'

        provider_names_split = provider.split(' ')
        new_provider_split_list = []
        for part_of_name in provider_names_split:
            if ',' in part_of_name:
                mini_list = part_of_name.split(',')
                part_of_name = mini_list[0]
            if part_of_name != '':
                new_provider_split_list.append(part_of_name)

        last_item = new_provider_split_list[-1]

        # This first sorts if there is a title in the name as default
        # These are the titles that came up:
        # RN, PA-C, DO, MD, DPM, CRNP, RD
        if last_item == 'RN':
            category = 'Virtual Nursing'
        if last_item == 'PA-C':
            category = 'Doctor Consult'
        if last_item == 'DO':
            category = 'Doctor Consult'
        if last_item == 'MD':
            category = 'Doctor Consult'
        if last_item == 'DPM':
            category = 'Doctor Consult'
        if last_item == 'CRNP':
            category = 'Doctor Consult'
        if last_item == 'RD':
            category = 'Dietary'
        
        # Categories: 
        # - Doctor Consult
        # - Virtual Nursing 
        # - Dietary 
        # - Case Manager
        # - Chaplain
        # - IT / Support
        # - Misc
        
        if len(new_provider_split_list) == 2:
            comparable_name = new_provider_split_list[0] + ' ' + new_provider_split_list[1]

            if comparable_name in use_cases_dict: 
                category = use_cases_dict.get(comparable_name)
        
        if len(new_provider_split_list) == 3:
            comparable_name = new_provider_split_list[0] + ' ' + new_provider_split_list[1] + ' ' + new_provider_split_list[2]

            if comparable_name in use_cases_dict:
                category = use_cases_dict.get(comparable_name)
            else: 
                comparable_name = new_provider_split_list[0] + ' ' + new_provider_split_list[1]
                if comparable_name in use_cases_dict:
                    category = use_cases_dict.get(comparable_name)
            
        if len(new_provider_split_list) == 4:
            comparable_name = new_provider_split_list[0] + ' ' + new_provider_split_list[1] + ' ' + new_provider_split_list[2] + ' ' + new_provider_split_list[3] 

            if comparable_name in use_cases_dict:
                category = use_cases_dict.get(comparable_name)
            else: 
                comparable_name = new_provider_split_list[0] + ' ' + new_provider_split_list[1] + ' ' + new_provider_split_list[2]
                if comparable_name in use_cases_dict:
                    category = use_cases_dict.get(comparable_name)
                else: 
                    comparable_name = new_provider_split_list[0] + ' ' + new_provider_split_list[2]
                    if comparable_name in use_cases_dict:
                        category = use_cases_dict.get(comparable_name)
                    else: 
                        comparable_name = new_provider_split_list[0] + ' ' + new_provider_split_list[1]
                        if comparable_name in use_cases_dict:
                            category = use_cases_dict.get(comparable_name)
        

        row.append(category)
    
    use_case_table[0][8] = 'Use Case'


    return use_case_table



def add_specialty(table):
    specialty_table = table

    specialty_dict = get_key_values_for_dict('specialties_key_values.csv')

    for row in specialty_table: 
        provider = row[1]
        use_case = row[8]

        specialty = ''

        if use_case == 'Doctor Consult':

            specialty = 'Misc'

            provider_names_split = provider.split(' ')
            new_provider_split_list = []
            for part_of_name in provider_names_split:
                if ',' in part_of_name:
                    mini_list = part_of_name.split(',')
                    part_of_name = mini_list[0]
                if part_of_name != '':
                    new_provider_split_list.append(part_of_name)
                
            
            if len(new_provider_split_list) == 2:
                comparable_name = new_provider_split_list[0] + ' ' + new_provider_split_list[1]

                if comparable_name in specialty_dict: 
                    specialty = specialty_dict.get(comparable_name)
        
            if len(new_provider_split_list) == 3:
                comparable_name = new_provider_split_list[0] + ' ' + new_provider_split_list[1] + ' ' + new_provider_split_list[2]

                if comparable_name in specialty_dict:
                    specialty = specialty_dict.get(comparable_name)
                else: 
                    comparable_name = new_provider_split_list[0] + ' ' + new_provider_split_list[1]
                    if comparable_name in specialty_dict:
                        specialty = specialty_dict.get(comparable_name)

            if len(new_provider_split_list) == 4:
                comparable_name = new_provider_split_list[0] + ' ' + new_provider_split_list[1] + ' ' + new_provider_split_list[2] + ' ' + new_provider_split_list[3] 

                if comparable_name in specialty_dict:
                    specialty = specialty_dict.get(comparable_name)
                else: 
                    comparable_name = new_provider_split_list[0] + ' ' + new_provider_split_list[1] + ' ' + new_provider_split_list[2]
                    if comparable_name in specialty_dict:
                        specialty = specialty_dict.get(comparable_name)
                    else: 
                        comparable_name = new_provider_split_list[0] + ' ' + new_provider_split_list[2]
                        if comparable_name in specialty_dict:
                            specialty = specialty_dict.get(comparable_name)
                        else: 
                            comparable_name = new_provider_split_list[0] + ' ' + new_provider_split_list[1]
                            if comparable_name in specialty_dict:
                                specialty = specialty_dict.get(comparable_name)
        

        row.append(specialty)
                
    specialty_table[0][9] = 'Specialty'

    return specialty_table            




# Define and instantiate the app
app = App()
app.mainloop()
```

