<template lang="pug">
    .container.my-5.mx-auto
        h1.mb-5
            | Debt Resolver
            |
            a.btn.btn-outline-primary.float-right(href="https://lixquid.com").
                lixquid.com
        SplitterTable.mb-3(@debt-export="debts = $event")

        .card
            .card-header Debt Solver
            .card-body
                .text-right.mb-3
                    button.btn.btn-outline-danger(
                        @click.passive="debts = [{debtor: '', amount: 0, creditor: ''}]"
                    ) Reset
                .row(v-for="(r, i) in debts", :key="i")
                    .input-group.col
                        input.form-control(
                            type="text"
                            placeholder="Debtor"
                            v-model="r.debtor"
                        )
                        .input-group-append
                            button.btn.btn-secondary.dropdown-toggle(data-toggle="dropdown")
                            .dropdown-menu
                                a.dropdown-item(
                                    v-for="(n, i) in allNames"
                                    :key="i"
                                    href="#"
                                    @click.prevent="r.debtor = n"
                                ) {{n}}
                    span.debt-text owes
                    .col-3
                        input.form-control(
                            type="number"
                            v-model.number="r.amount"
                            min="0"
                            step="0.01"
                        )
                    span.debt-text to
                    .input-group.col
                        input.form-control(
                            type="text"
                            placeholder="Creditor"
                            v-model="r.creditor"
                        )
                        .input-group-append
                            button.btn.btn-secondary.dropdown-toggle(data-toggle="dropdown")
                            .dropdown-menu
                                a.dropdown-item(
                                    v-for="(n, i) in allNames"
                                    :key="i"
                                    href="#"
                                    @click.prevent="r.debtor = n"
                                ) {{n}}
                    button.btn.btn-outline-danger.remove-debt(
                        @click.passive="debts.splice(i, 1)"
                        :disabled="debts.length === 1"
                    ) #[span.fas.fa-times]
                button.btn.btn-success.mt-3.w-100(
                    @click.passive="debts.push({debtor:'', amount: 0, creditor: ''})"
                ) #[span.fas.fa-plus] New Debt
            .card-body.border-top
                ul.mb-0
                    li(v-if="resolvedDebts.length === 0").
                        No-one owes anything!
                    li(
                        v-for="(d, i) in resolvedDebts"
                        :key="i"
                    ) {{d.debtor}} gives {{d.amount}} to {{d.creditor}}

</template>

<script lang="ts">
import { defineComponent, ref, computed } from "@vue/composition-api";
import SplitterTable from "./components/SplitterTable.vue";
import { chain } from "lodash";
import { IDebt } from "./types";

export default defineComponent({
    components: {
        SplitterTable
    },
    setup() {
        const debts = ref<IDebt[]>([
            {
                debtor: "",
                amount: 0,
                creditor: ""
            }
        ]);

        const allNames = computed(() =>
            chain(debts.value)
                .flatMap(d => [d.debtor, d.creditor])
                .uniq()
                .filter(v => !!v)
                .value()
        );

        const resolvedDebts = computed(() => {
            // Calculate balance
            const balances: { [name: string]: number } = {};
            debts.value.forEach(e => {
                if (!e.debtor || !e.creditor) {
                    return;
                }
                balances[e.debtor] = balances[e.debtor] ?? 0;
                balances[e.creditor] = balances[e.creditor] ?? 0;
                balances[e.debtor] -= e.amount;
                balances[e.creditor] += e.amount;
            });

            // Assign values
            const transactions: IDebt[] = [];

            Object.keys(balances).forEach(debtor => {
                if (balances[debtor] >= 0) {
                    return;
                }
                Object.keys(balances).forEach(creditor => {
                    if (balances[creditor] <= 0) {
                        return;
                    }
                    if (-balances[debtor] > balances[creditor]) {
                        // creditor out of credit
                        transactions.push({
                            debtor,
                            creditor,
                            amount: balances[creditor]
                        });
                        balances[debtor] += balances[creditor];
                        balances[creditor] = 0;
                    } else {
                        // debtor debt cleared
                        transactions.push({
                            debtor,
                            creditor,
                            amount: -balances[debtor]
                        });
                        balances[creditor] += balances[debtor];
                        balances[debtor] = 0;
                    }
                });
            });

            return transactions;
        });

        return {
            debts,
            allNames,
            resolvedDebts
        };
    }
});
</script>

<style lang="stylus" scoped>
.debt-text
    line-height 2

.remove-debt
    margin-right 15px
</style>
