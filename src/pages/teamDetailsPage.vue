<template>
    <div id="team-page-div">

        <!----------  Header  ---------->

        <b-img style="position:absolute; left: 80%;top:8%;" id="team-logo" :src="teamLogo" ></b-img> 
        <h1 class="team-page-hade"> {{teamName}} </h1>


        <!----------  Team's Squad  ---------->

        <div  class="container my-container-css" id="squad-div">

            <h2 class="team-page-title" id="squad-head"> Team's Squad : </h2>
            <div class="row align-items-start my-row-css">
                <player-information 
                    v-for="player in this.squadInformation" :key="player.playerID" :player="player">
                </player-information>
            </div>
        </div>


        <!----------  Team's Future Matches  ---------->

            <!----------  Future Table  ---------->
        
        <div  class="container my-container-css" id="future-matches-div">
            <h2 class="team-page-title" id="future-match-head"> Team's Future Matches : </h2>

            <b-table id="future-matches-table" v-if="rows_Future"
                :items="teamFutureMatches"
                hover
                bordered
                outlined
                no-border-collapse
                head-variant="light"
                table-variant="light"
                >

                <template #head(refereeInformation)>
                    Referee Full Name
                </template>

                <template #cell(refereeInformation)="{ value: refereeInformation }">
                    {{refereeFullName(refereeInformation)}}
                </template>
            </b-table>

            <!----------  Pagination  ---------->

            <b-pagination class="matches-pagination" v-if="rows_Future > 5"
                v-model="currentPage_Future"
                :total-rows="rows_Future"
                :per-page="perPage"
                aria-controls="matches-table">
            </b-pagination>

            <!---------- No Matches Message  ---------->

            <div class="container my-container-css" v-if="rows_Future==0" >
                <h3 style="text-align:center;" >
                    There Are No Future Matches
                </h3>
            </div>
        </div>


        <!----------  Team's Past Matches  ---------->

            <!----------  Past Table  ---------->

        
        <div  class="container my-container-css" id="past-matches-div">
            <h2 class="team-page-title" id="past-match-head"> Team's Past Matches : </h2>

            <b-table id="past-matches-table" v-if="rows_Past"
                :items="teamPastMatches"
                :fields="fields"
                hover
                bordered
                outlined
                no-border-collapse
                head-variant="light"
                table-variant="light"
                >

                <template #head(refereeInformation)>
                    Referee Full Name
                </template>

                <template #cell(matchScore)="row" >
                    <b>{{ row.item.localTeamScore }}</b>  :  <b>{{ row.item.visitorTeamScore }}</b>
                </template>

                <template #cell(refereeInformation)="{ value: refereeInformation }">
                    {{refereeFullName(refereeInformation)}}
                </template>

                <template #cell(eventsLog)="row">
                    <b-button size="sm" @click="row.toggleDetails" class="mr-2" v-if="row.item.eventsLog.length">
                        {{ row.detailsShowing ? 'Hide' : 'Show'}} Details
                    </b-button>
                </template>

                <template #row-details="row">
                    <b-card>
                        <events-log-preview 
                            :eventsLog="row.item.eventsLog">
                        </events-log-preview>

                        <b-button size="sm" @click="row.toggleDetails" class="mr-2">
                            Hide Details
                        </b-button>
                    </b-card>
                </template>

            </b-table>

            <!----------  Pagination  ---------->

            <b-pagination class="matches-pagination" v-if="rows_Past > 5"
                v-model="currentPage_Past"
                :total-rows="rows_Past"
                :per-page="perPage"
                aria-controls="matches-table">
            </b-pagination>

            <!---------- No Matches Message  ---------->

            <div class="container my-container-css" v-if="rows_Past==0" >
                <h3 style="text-align:center;" >
                    There Are No Past Matches
                </h3>
            </div>
        </div>

    </div>
</template>


<script>

import PlayerInformation from '../components/PlayersSearchPreview.vue';
import EventsLogPreview from '../components/matches/match_eventsLogPreview.vue';

export default {
    name: "TeamDetails",

    components: {
        PlayerInformation,
        EventsLogPreview,
    },

    data() {
        return {

            teamName: "",
            teamLogo: "",
            teamSquadIDs: [],
            squadInformation: [],
            teamFutureMatches: [],
            teamPastMatches: [],
            fields: [ 
                "matchID", "matchDate", "localTeamName", "visitorTeamName", "venueName", "matchScore", "refereeInformation", "eventsLog"
            ],

            perPage : 5,
            currentPage_Future : 1,
            currentPage_Past : 1,

        }
    },
    methods: {
        extractPlayersInfo() {

            var players = JSON.parse(localStorage.getItem("playersInfo"));
            players.map( (player) => {
                if ( this.teamSquadIDs.includes(player.playerShortInfo.playerID) ){
                    this.squadInformation.push(player.playerShortInfo);
                }
            });
        },
        refereeFullName(refereeInformation) {
            if ( Object.keys(refereeInformation).length==0 ){
                return "  -  ";
            }
            return refereeInformation.firstname + " " + refereeInformation.lastname;
            
        },
    },
    computed: {
        rows_Future() {
            return this.teamFutureMatches.length;
        },
        rows_Past() {
            return this.teamPastMatches.length;
        },
    },
    mounted() {

        /* --------  Basic Teams Data  -------- */
        
        this.teamName = this.$route.params.teamName;
        var teamInfo = JSON.parse(localStorage.getItem("teamsInfo")); 
        teamInfo.map( (team) => {
            if ( team.teamName == this.teamName ) {
                this.teamLogo = team.teamLogo;
                this.teamSquadIDs = team.teamSquad;
            }
        });

        /* --------  Extract Players Information  -------- */

        this.extractPlayersInfo();

        /* --------  Teams Matches  -------- */

        var curStageFutureMatches = [];
        var curStagePastMatches = [];
        if ( localStorage.getItem("CurrentStageMatchesFutureMatches") ){
            var futureMatches = JSON.parse(localStorage.getItem("CurrentStageMatchesFutureMatches"));
            futureMatches.map(( match ) => {
                curStageFutureMatches.push({
                    matchID: match.matchID,
                    matchDate: match.matchDate,
                    localTeamName: match.localTeamName,
                    visitorTeamName: match.visitorTeamName,
                    venueName: match.venueName,
                    refereeInformation: match.refereeInformation
                })
            })
        }
        if ( localStorage.getItem("CurrentStageMatchesPastMatches") ){
            curStagePastMatches = JSON.parse(localStorage.getItem("CurrentStageMatchesPastMatches"));
        }
        
        var futureMatchesToAdd = [];
        var pastMatchesToAdd = [];

        curStageFutureMatches.map((match) => {
            if ( match.localTeamName == this.teamName || match.visitorTeamName == this.teamName ){
                futureMatchesToAdd.push(match);
            }
        });

        curStagePastMatches.map((match) => {
            if ( match.localTeamName == this.teamName || match.visitorTeamName == this.teamName ){
                pastMatchesToAdd.push(match);
            }
        });

        this.teamFutureMatches.push(...futureMatchesToAdd);
        this.teamPastMatches.push(...pastMatchesToAdd);
    }
}

</script>


<style>

/* --------  Page Div  -------- */

#team-page-div {
    padding-bottom: 50px;
    background: rgba(253, 245, 209, 0.322);
}

/* --------  Page Header  -------- */
.team-page-hade {
    display: flex;
    flex-direction: column;
    justify-content: center;
    text-align: center;
    color: rgb(0, 0, 0);
    font-weight: bold;
    font-size: 100px;
    padding-top: 20px;
}
.team-page-title {
    display: flex;
    flex-direction: column;
    justify-content: center;
    text-align: center;
    color: rgb(255, 255, 255);
    font-weight: bold;
}

#team-logo {
    position: absolute;
    left: 35px;
    margin-top: 0px;
}


/* --------  Squad Div  -------- */

#squad-div {
    margin-top: 60px ;
}

#squad-head {
    margin-top: 0px;
    padding-top: 15px;
    padding-bottom: 5px;
}

.container {
    margin: auto;
    max-width: 90%;
}

.my-container-css {
    overflow: scroll;
    overflow-x: hidden;
    background-color: #293241e0;
    max-height:56.4vmin;
    border-radius: 15px;
}

.my-row-css {
    padding: 40px;
}


/* -------- Future + Past Matches Div  -------- */

#future-matches-div, #past-matches-div {
    margin: auto;
    width: 90%;
    padding: 15px;
    margin-top: 20px;
}

#past-match-head, #future-match-head {
    margin-top: 0px;
    padding-bottom: 5px;
}

#future-matches-table, #past-matches-table {
    text-align: center;
    align-items: center;
}

h3 {
    color: white;
}

/* -------- Pagination  -------- */

.matches-pagination {
    margin-left: 45%; 
    margin-right: 45%;
}

</style>