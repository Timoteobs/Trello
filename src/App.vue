<template>
  <v-app>
    <v-app-bar app color="primary" dark> </v-app-bar>

    <v-main>
      <div class="board">
        <div v-for="lane in itemLane" :key="lane.id" class="lane">
          <h3 class="lane-title">{{ lane.title }}</h3>
          <div class="input-Card">
            <v-text-field
              label="Texto"
              v-model="lane.cardName"
              hide-details="auto"
              required
            ></v-text-field>
            <v-btn
              depressed
              color="primary"
              @click="newCard(lane.id, lane.cardName)"
            >
              Adicionar
            </v-btn>
          </div>
          <Container
            group-name="trello"
            @drag-start="start(lane.id, $event)"
            @drop="end(lane.id, $event)"
            :get-child-payload="get"
            :drop-placeholder="{ className: 'placeholder' }"
          >
            <Draggable v-for="card in lane.cards" :key="card.id">
              <Card
                :iconClick="
                  () => {
                    dialog = true;
                  }
                "
                >{{ card.text }}
                <v-row justify="center">
                  <v-dialog v-model="dialog" persistent max-width="400">
                    <v-card>
                      <v-card-title>
                        <span class="headline">Ediar card</span>
                      </v-card-title>
                      <v-card-text>
                        <v-text-field
                          label="Texto"
                          v-model="card.text"
                          hide-details="auto"
                          class="card-text"
                        ></v-text-field>
                      </v-card-text>
                      <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn
                          color="red darken-1"
                          text
                          @click="
                            editCard(lane.id, card.id, card.text, 'delete')
                          "
                        >
                          Excluir
                        </v-btn>
                        <v-btn
                          color="green darken-1"
                          text
                          @click="dialog = false"
                        >
                          Cancelar
                        </v-btn>
                        <v-btn
                          color="green darken-1"
                          text
                          @click="editCard(lane.id, card.id, card.text, 'edit')"
                        >
                          Salvar
                        </v-btn>
                      </v-card-actions>
                    </v-card>
                  </v-dialog>
                </v-row>
              </Card>
            </Draggable>
          </Container>
        </div>
        <div class="input-lane">
          <v-text-field
            label="Título da lista"
            v-model="itemName"
          ></v-text-field>
          <v-btn depressed color="primary" @click="newLane"> Nova lista </v-btn>
        </div>
      </div>
    </v-main>
  </v-app>
</template>

<script>
import Card from "./components/Card";
import { Container, Draggable } from "vue-smooth-dnd";
export default {
  name: "App",
  components: {
    Card,
    Container,
    Draggable,
  },
  data: () => ({
    itemName: "",
    dialog: false,
    itemLane: [
      {
        id: 1,
        title: "To Do",
        cards: [{ id: 1, text: "Estudar Vue" }],
        cardName: "",
      },
      {
        id: 2,
        title: "Doing",
        cards: [],
        cardName: "",
      },
      {
        id: 3,
        title: "Done",
        cards: [],
        cardName: "",
      },
    ],
    draggingCard: {
      lane: "",
      index: -1,
      cardData: {},
    },
  }),
  methods: {
    newLane() {
      if (this.itemName !== "") {
        this.itemLane.push({
          id: Math.random(),
          title: this.itemName,
          cards: [],
          cardName: "",
        });
        this.itemName = "";
      } else {
        alert("Insira o Título da lista, para continuar.");
      }
    },
    newCard(id, cardName) {
      if (cardName !== "") {
        for (let index = 0; index < this.itemLane.length; index++) {
          if (id == this.itemLane[index].id) {
            this.itemLane[index].cards.push({
              id: Math.random(),
              text: cardName,
            });
            this.itemLane[index].cardName = "";
          }
        }
      } else {
        alert("Insira o texto, para continuar.");
      }
    },
    editCard(idLane, idCard, cardText, action) {
      for (let index = 0; index < this.itemLane.length; index++) {
        if (idLane == this.itemLane[index].id) {
          for (
            let indexCard = 0;
            indexCard < this.itemLane[index].cards.length;
            indexCard++
          ) {
            if (action == "edit") {
              if (this.itemLane[index].cards[indexCard].id == idCard) {
                this.itemLane[index].cards[indexCard].text = cardText;
                this.dialog = false;
              }
            } else {
              this.itemLane[index].cards.splice(
                this.itemLane[index].cards.indexOf(idCard),
                1
              );
              this.dialog = false;
            }
          }
        }
      }
    },
    start(lane, dragResult) {
      const { payload, isSource } = dragResult;

      for (let index = 0; index < this.itemLane.length; index++) {
        if (lane == this.itemLane[index].id) {
          if (isSource) {
            this.draggingCard = {
              lane,
              index: payload.index,
              cardData: {
                ...this.itemLane[index].cards[payload.index],
              },
            };
          }
        }
      }
    },
    end(lane, dropResult) {
      const { removedIndex, addedIndex } = dropResult;

      if (lane === this.draggingCard.lane && removedIndex === addedIndex) {
        return;
      }

      if (removedIndex !== null) {
        for (let index = 0; index < this.itemLane.length; index++) {
          if (this.itemLane[index].id == lane) {
            this.itemLane[index].cards.splice(removedIndex, 1);
          }
        }
      }

      if (addedIndex !== null) {
        for (let index = 0; index < this.itemLane.length; index++) {
          if (this.itemLane[index].id == lane) {
            this.itemLane[index].cards.splice(
              addedIndex,
              0,
              this.draggingCard.cardData
            );
          }
        }
      }
    },
    get(index) {
      return {
        index,
      };
    },
  },
};
</script>

<style>
.board {
  display: flex;
  justify-content: flex-start;
  margin: 1.2rem 0.8rem;
  align-items: flex-start;
}
.lane {
  background: rgb(216, 214, 214);
  width: 23rem;
  border-radius: 10px;
  box-shadow: 0 0.1rem 0.2rem 0 rgba(33, 33, 33, 0.1);
  margin: 0 0.8rem;
  padding: 0 0.7rem;
}
.lane-title {
  padding: 0.8rem 0.5rem;
  margin-bottom: 0.6ren;
}
.input-Card {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 30px;
}
.input-lane {
  display: flex;
  justify-content: center;
}
.card-text {
  margin-top: 50px;
}
.placeholder {
  background: rgba(33, 33, 33, 0.08);
  border-radius: 0.4rem;
  transform: scaleY(0.85);
  transform-origin: 0% 0%;
}
</style>
