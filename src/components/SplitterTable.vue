<template lang="pug">
    .card
        .card-header Splitter Table
        .card-body
            .float-right.mb-3
                button.btn.btn-outline-info(
                    data-toggle="modal"
                    data-target="#helpModal"
                ) Help
                | 
                button.btn.btn-outline-danger(
                    @click.passive="names = []; lines = []"
                ) Reset
            table.table.table-bordered
                thead
                    tr
                        th(width="20%")
                        th(v-for="(n, i) in names", :key="i")
                            .input-group
                                input.form-control(
                                    v-model="names[i]"
                                )
                                .input-group-append
                                    button.btn.btn-outline-danger(
                                        @click.passive="names.splice(i, 1)"
                                        title="Remove Person"
                                    ) &times;
                        th(width="1px")
                            button.btn.btn-outline-success(
                                @click.passive="names.push('Person ' + names.length)"
                                title="Add Person"
                            ) #[span.fas.fa-plus]
                tbody
                    tr(v-for="(l, i) in lines", :key="i")
                        th
                            .input-group
                                input.form-control(
                                    placeholder="Name"
                                    v-model="l.name"
                                )
                                input.form-control(
                                    type="number"
                                    v-model.number="l.amount"
                                    min="0"
                                    step="0.01"
                                )
                                .input-group-append
                                    button.btn.btn-outline-secondary.dropdown-toggle(
                                        data-toggle="dropdown"
                                    ) {{l.owner}}
                                    .dropdown-menu
                                        a.dropdown-item(
                                            v-for="(n, i) in names"
                                            :key="i"
                                            href="#"
                                            @click.prevent="l.owner = n"
                                        ) {{n}}
                        td(v-for="(n, tdi) in names", :key="tdi")
                            input.form-control(
                                type="number"
                                v-model.number="l.ratios[n]"
                                min="0"
                                step="0.01"
                            )
                        td
                            button.btn.btn-outline-danger(
                                @click.passive="lines.splice(i, 1)"
                            ) #[span.fas.fa-times]
            
            button.btn.btn-success.mt-3.w-100(
                @click.passive="lines.push({name: '', owner: '', amount: 0, ratios: {}})"
                :disabled="names.length < 2"
            ) #[span.fas.fa-plus] New Line Item
            small.text-info(v-if="names.length < 2").
                Add names using the plus button before adding line items.
            button.btn.btn-primary.mt-3(
                v-if="lines.length"
                @click.passive="exportLines"
            ) Export to Debt Solver

        .modal.fade#helpModal(tabindex=-1)
            .modal-dialog
                .modal-content
                    .modal-header
                        h5.modal-title Debt Splitter Help
                        button.close(data-dismiss="modal") &times;
                    .modal-body
                        :markdown-it
                            The debt splitter can be used to tally up all debts incurred during an event.

                            1. Use the **Add Person** button to add the number of columns equal to the number of people participating.
                            2. Use the **New Line item** button to add a new entry for an expense.
                            3. In the left-most column, add in what the expense was for, how much it was, and who paid it.
                            4. Under each person's name, add in a number ratio for how much of that expense they are responsible for.
                                - For example, if everyone is equally responsible, put a `1` for all entries.
                                - If you know the exact values of the items people bought, put in those values.
                            5. When all line items are added, press **Export to Debt Solver** to automatically export the values ot the debt solver.

                            ##### Example

                            Alice, Bob, and Charlie go on a camping trip. Alice drives them all, Bob pays for the site fee, and Charlie pays for the food.

                            Everyone is equally responsible for fuel costs, so that is equally spread amongst everyone.

                            Charlie brought a double-wide tent, so he should pay twice as much as Alice and Bob for the site rental.

                            Everyone bought one meal item each, so exact values can be used.

                        button.btn.btn-primary(
                            @click.passive="setExample"
                            data-dismiss="modal"
                        ) Show me what it looks like
</template>

<script lang="ts">
import { defineComponent, ref } from "@vue/composition-api";
import { sum } from "lodash";

interface ILine {
    name: string;
    owner: string;
    amount: number;
    ratios: { [name: string]: number };
}

export default defineComponent({
    setup(_, instance) {
        const names = ref<string[]>([]);
        const lines = ref<ILine[]>([]);

        function exportLines() {
            instance.emit(
                "debt-export",
                lines.value.flatMap(l => {
                    const total = l.amount / sum(Object.values(l.ratios));
                    return Object.keys(l.ratios)
                        .filter(n => l.ratios[n])
                        .map(n => ({
                            debtor: n,
                            creditor: l.owner,
                            amount: l.ratios[n] * total
                        }));
                })
            );
        }

        function setExample() {
            names.value = ["Alice", "Bob", "Charlie"];

            lines.value = [
                {
                    name: "Fuel",
                    amount: 60,
                    owner: "Alice",
                    ratios: {
                        Alice: 1,
                        Bob: 1,
                        Charlie: 1
                    }
                },
                {
                    name: "Rental",
                    amount: 40,
                    owner: "Bob",
                    ratios: {
                        Alice: 1,
                        Bob: 1,
                        Charlie: 2
                    }
                },
                {
                    name: "Food",
                    amount: 34.97,
                    owner: "Charlie",
                    ratios: {
                        Alice: 13.99,
                        Bob: 12.99,
                        Charlie: 7.99
                    }
                }
            ];
        }

        return {
            names,
            lines,
            exportLines,
            setExample
        };
    }
});
</script>
