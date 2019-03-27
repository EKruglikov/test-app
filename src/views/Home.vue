<template>
  <div class="home">

    <Header></Header>

    <!--Витрина-->
    <section class="section">
      <div class="container">
        <h2 class="section__title">Витрина</h2>

        <!-- Фильтры -->
        <div class="filters">
          <select class="ui-select" v-model="filter.subject">
            <option value="" selected>Все предметы</option>
            <option
                v-for="(subject, index) in subjects"
                :key="index"
                :value="subject">
              {{ subject }}
            </option>
          </select>
          <select class="ui-select" v-model="filter.genre">
            <option value="" selected>Все жанры</option>
            <option
                v-for="(genre, index) in genres"
                :key="index"
                :value="genre">
              {{ genre }}
            </option>
          </select>
          <select class="ui-select" v-model="filter.grade">
            <option value="" selected>Все классы</option>
            <!--Классы числами или все существующие в списке {grades}-->
            <option
                v-for="(grade, index) in 11"
                :key="index"
                :value="grade">
              {{ grade }}
            </option>
          </select>
          <input class="ui-input" type="text" placeholder="Поиск" v-model="filter.searchString">
          <button class="filters__search" type="submit" @click="filterItems"></button>
        </div>

        <div class="type-price">
          <label for="typePrice" class="ui-toggle">Показать цену в бонусах <span :class="{'is-active': !normalPrice}"></span> </label>
          <input type="checkbox" id="typePrice" @change="normalPrice =! normalPrice">
        </div>

        <!-- Витрина -->
        <div class="showcase">

          <!--Item-->
          <div class="showcase-item" v-for="item in filteredItems" :key="item.courseId">
            <div class="showcase-item__image"><img src="https://www.imumk.ru/svc/coursecover/81" alt=""></div>
            <div class="showcase-item__info">
              <div class="showcase-item__name">{{ item.subject }}</div>
              <div class="showcase-item__grade">{{ getGrade(item.grade) }}</div>
              <div class="showcase-item__genre">{{ item.genre }}</div>
              <a :href="item.shopUrl" class="showcase-item__more">Подробнее</a>
              <div class="showcase-item__price">
                <span>{{ getPrice(item) }} рублей</span>
              </div>
            </div>
          </div>

        </div>

      </div>
    </section>

  </div>
</template>

<script>
// @ is an alias to /src
import Header from '@/components/Header';
export default {
  name: 'home',
  components: { Header },
  data () {
    return {
      filter: {subject: '', genre: '', grade: '', searchString: ''},
      normalPrice: true,
      items: [],
      subjects: [],
      genres: [],
      grades: []
    }
  },
  computed: {
    filteredItems () {
      return this.items
          .filter((item) => { return this.filter.subject == '' || item.subject  == this.filter.subject })
          .filter((item) => { return this.filter.genre == '' || item.genre  == this.filter.genre })
          .filter((item) => { return this.filter.grade == '' || item.grade  == this.filter.grade })
          .filter((item) => {
            if (this.filter.searchString == '' || this.searchByString(item)) { return item; }
          })
    }
  },
  methods: {
    async getItems () {
      let getItems = await axios({
        method: 'post',
        url: 'http://krapipl.imumk.ru:8082/api/mobilev1/update',
        data: {data: ''}
      });

      this.items = getItems.data.items;
      this.subjectsList();
      this.genresList();
      this.gradesList();

      console.log('ITEMS: ', getItems.data);
    },

    filterItems () {
      this.items = this.items
          .filter((item) => { return this.filter.subject == '' || item.subject  == this.filter.subject })
          .filter((item) => { return this.filter.genre == '' || item.genre  == this.filter.genre })
          .filter((item) => { return this.filter.grade == '' || item.grade  == this.filter.grade })
          .filter((item) => { return (this.filter.searchString == '' || this.searchByString(item)) })
    },

    searchByString (item) {
        if (
            item.title.toLocaleLowerCase().match(this.filter.searchString.toLocaleLowerCase()) ||
            item.genre.toLocaleLowerCase().match(this.filter.searchString.toLocaleLowerCase()) ||
            item.description.toLocaleLowerCase().match(this.filter.searchString.toLocaleLowerCase()) ||
            item.subject.toLocaleLowerCase().match(this.filter.searchString.toLocaleLowerCase())
        ) {
          return true;
        } else {
          return false;
        }
    },

    subjectsList() {
      this.items.forEach((item, index) => {
        if (!this.subjects.includes(item.subject)) {
          this.subjects.push(item.subject);
        }
      });
    },
    genresList() {
      this.items.forEach((item, index) => {
        if (!this.genres.includes(item.genre)) {
          this.genres.push(item.genre);
        }
      });
    },
    gradesList() {
      this.items.forEach((item, index) => {
        let intGrade = parseInt(item.grade);
        if (!this.grades.includes(intGrade)) {
          if (!item.grade.match(/\;/)){
            this.grades.push(intGrade);
          } else {
            let grades = item.grade.split(';');
            for(let grade of grades) {
              let intSubGrade = parseInt(grade);
              if (!this.grades.includes(intSubGrade)) {
                this.grades.push(intSubGrade);
              }
            }
          }
        }
      });
    },

    getGrade(_grade){
      if (_grade.match(/\;/)) {
        let str = _grade.split(';');
        return str[0] + '-' + str[str.length - 1] + ' классы';
      } else {
        return _grade + ' класс';
      }
    },
    getPrice(_item) {
      return this.normalPrice ? _item.price : _item.priceBonus;
    }
  },
  mounted() {
    this.getItems();
  }
}
</script>