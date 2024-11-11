<script setup>

</script>

<template>
  <textarea name="plsqlcode" id="plsqlcode" class="full" rows="30" v-model="plsqlcode" @input="doit()"></textarea>

<div>
  Items to submit:
  <textarea id="submit" class="full" v-model="items_to_submit"></textarea>
  <br>
  Items to return:
  <textarea id="return" class="full" v-model="items_to_return"></textarea>
</div>

  <button @click="doit()" class="btn btn-primary">Convert</button>

  <table>
    <thead>
      <tr>
        <th>Naziv</th>
        <th>Submit</th>
        <th>Return</th>
      </tr>
    </thead>

    <tbody>
      <tr v-for="item in items">
        <td>{{item.name}}</td>
        <td><input type="checkbox" v-model="item.submit" /></td>
        <td><input type="checkbox" v-model="item.return" /></td>
      </tr>
    </tbody>
  </table>


</template>

<style scoped>
  .half{
    width: calc(50%);
  }
  .full{
    width: 100%;
  }
</style>

<script>

  const re = new RegExp(":[A-Za-z0-9_]+", 'g');

  export default {
    data() {
      return {
        items: [],
        plsqlcode: `pisi_is_bf('BB_UPL_ISPL', :UPLATNICA_ISPLATNICA);

  if :BLAGAJNA is not null then
          
      begin
        select max(dnevnik_blag) into :BROJ_DNEVNIKA from blagajne
          where vr_sifra_vr = :BLAGAJNA
            and datdok_blag = :DATUM_DOKUMENTA
            and godina_blag = to_number(to_char(to_date(:DATUM_DOKUMENTA),'yyyy'));
      exception
          when no_data_found then   
          null;
          when others then
              :ALERT := '33e23 ' || SQLERRM;
      end;

      if :BROJ_DNEVNIKA is null then
          :BROJ_DNEVNIKA := 1;
          :BROJ_DOKUMENTA := 0;	
      end if;

      begin

          select max(brdok_blag) into :BROJ_DOKUMENTA from blagajne
              where vr_sifra_vr = :BLAGAJNA
              and godina_blag = to_number(to_char(to_date(:DATUM_DOKUMENTA),'yyyy'))
              and	upl_ispl_blag = :UPLATNICA_ISPLATNICA;

          if :BROJ_DOKUMENTA is null then
              :BROJ_DOKUMENTA := 0;
          end if;      
      exception
          when others then
              :ALERT := '22e22 ' || SQLERRM;
      end;


      PKG_BRZA_BLAGAJNA.p_saldiraj(
          :BLAGAJNA,
          :DATUM_DOKUMENTA,

          :POCETNI_SALDO,
          :PRIMICI,
          :IZDACI,
          :SALDO
      );

  end if;
  
  if :UPLATNICA_ISPLATNICA = 'U' then
      :strana_bsh := 'P';
  else
      :strana_bsh := 'D';
  end if;	`,
      }
    },

    methods: {
      doit(){
        const matches = this.plsqlcode.matchAll(re);

        this.items = [];

        for (const match of matches) {
            console.log(
                `Found ${match[0]} start=${match.index} end=${
                match.index + match[0].length
                }.`,
            );
            if(!this.items.includes(match[0])){
              this.items.push(match[0])
            }
        }

        this.items = JSON.parse(JSON.stringify(this.items.map(item => {
          return {"name": item, "submit": true, "return": true};
        })));

      }
    },

    mounted() {
      this.doit();
    },

    computed: {
      items_to_submit() {
        return this.items.filter(item => item.submit).map(item => item.name.substr(1));
      },
      items_to_return() {
        return this.items.filter(item => item.return).map(item => item.name.substr(1));
      },
    }
  }
</script>