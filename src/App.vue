<script setup>
import { ref,onMounted } from 'vue';
import axios from 'axios';
import ExcelJS from 'exceljs';
import jsPDF from 'jspdf';
const lineNotifyToken = '1AisH0fcOhEvRJMnCfEtmEoDlYGr4NO7i9a7JN7gIDs';

  const pokemon = ref(null);
  const countPoke = 8;
  
  onMounted(async()=>{
    try {
      const PokemonArray = [];

      for(let i =1;i<=countPoke;i++){
        const response = await axios.get(`https://pokeapi.co/api/v2/pokemon/${i}`);

        PokemonArray.push(response.data);
      }
      pokemon.value = PokemonArray;
      console.log(pokemon.value);
    } catch (error) {
      console.log('error',error);
    }

    return{
      pokemon
    };

  });
  const sendLineNotification = async (message) => {
  try {
    const response = await axios.post(
      "https://notify-api.line.me/api/notify",
      'message=' + message, // ใส่ข้อมูล message เป็นส่วนหนึ่งของ URL-encoded data
      {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Authorization': 'Bearer ZTYS9P43IIEY4JmmsxBfzKS6WxlbFkOWlc4HCD2rrfa'
        },
      }
    );

    console.log('Notification sent to LINE successfully');
    console.log('Response:', response.data); // Log the response for additional information
  } catch (error) {
    console.error('Error sending notification to LINE', error.response?.status, error.response?.data);
  }
};



  const test = () =>{
    sendLineNotification('สวัสดีครับ/ค่ะ');
  }


  //excell
  const generateExcel = () => {

  
  const workbook = new ExcelJS.Workbook();
  const worksheet = workbook.addWorksheet('Pokemon');

  worksheet.columns = [
    { header: 'Name', key: 'name', width: 20 },
    { header: 'Image', key: 'image', width: 20 },
  ];

  pokemon.value.forEach((item) => {
    worksheet.addRow({ name: item.name, image: item.sprites.front_default });
  });

  workbook.xlsx.writeBuffer().then((data) => {
    const blob = new Blob([data], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });
    const link = document.createElement('a');
    link.href = URL.createObjectURL(blob);
    link.download = 'pokemon.xlsx';
    link.click();
  });
};

// PDF
const generatePDF = () => {
  const doc = new jsPDF();

  pokemon.value.forEach((item, index) => {
    const textX = 30;
    const textY = 20 + index * 50;
    const imageX = 30; // ตำแหน่งรูป x
    const imageY = 15 + index * 50; // ตำแหน่งรูป

    doc.text(`Name: ${item.name}`, textX, textY);
    doc.addImage(item.sprites.front_default, 'JPEG', imageX, imageY, 40, 40); // Adjust dimensions as needed
  });

  doc.save('pokemon.pdf');
};


</script>

<template>
  <div class="flex flex-col justify-center items-center pt-5">
    <h1>Hello Poke</h1>
    <div class="flex space-x-5 my-5">
      <button @click="test" class="px-2 w-20 rounded-md border-2 bg-rose-500">test</button>
      <button @click="generateExcel" class="px-2 w-20 rounded-md border-2 bg-rose-500">Excel</button>
      <button @click="generatePDF" class="px-2 w-20 rounded-md border-2 bg-orange-500">PDF</button>
    </div>
    <div v-if="pokemon" class="grid grid-cols-4 gap-5 w-full h-[10rem] px-5">
      <div v-for="item of pokemon" :key="item.name"
       class="border-2 rounded-md flex flex-col items-center justify-center ">
       <img :src="item.sprites.front_default" alt="imgPoke"/>
        {{ item.name }}
      </div>
    </div>
  </div>
</template>


