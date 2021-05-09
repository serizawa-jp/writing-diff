<script>
  import {
    Button,
    Modal,
    Column,
    Content,
    FileUploader,
    Grid,
    Row,
    TextArea,
  } from "carbon-components-svelte";
  import Header from "./components/Header.svelte";
  import Theme from "./components/Theme.svelte";
  import Papa from "papaparse";
  import { diffWords } from "diff";

  const parseFile = (rawFile) => {
    return new Promise((resolve) => {
      Papa.parse(rawFile, {
        header: false,
        complete: (results) => {
          resolve(results.data);
        },
      });
    });
  };

  let theme = "g10";
  let files = [];
  let questionCurrentIndex = 0;
  let questions = [];
  let isModalOpen = false;
  let isAnswerModalOpen = false;
  let currentAnswer = "";
  let answerDiff = [];
  let isShowQuestionModal = false;
  $: modalHeader = `Questions ${questionCurrentIndex + 1}/${questions?.length}`;
  $: _ = handleFiles(files);
  $: currentQuestion = questions[questionCurrentIndex];

  function openAnswerModal() {
    answerDiff = diff(currentAnswer, questions[questionCurrentIndex]?.answer);
    isAnswerModalOpen = true;
  }

  function closeAnswerModal() {
    currentAnswer = "";

    if (questionCurrentIndex !== questions.length - 1) {
      questionCurrentIndex++;
      return;
    }

    isModalOpen = false;
    questionCurrentIndex = 0;
  }

  function diff(old, newWord) {
    if (!old) old = "";
    if (!newWord) return [];
    return diffWords(old, newWord);
  }

  function isCorrect(d) {
    if (!d) return false;
    if (d?.length > 1) return false;

    const dd = d[0];
    if (dd?.hasOwnProperty("removed") || dd?.hasOwnProperty("added"))
      return false;

    return true;
  }

  async function handleFiles(files) {
    if (!files || files.length < 1) return null;
    const f = files[0];
    const data = await parseFile(f);
    const d = data
      .slice(1)
      .map((a) => {
        if (!a || a.length < 2) return null;

        const question = a[0].trim();
        const answer = a[1].trim();

        if (question === "" || answer === "") return null;

        return { question, answer };
      })
      .filter((a) => !!a);

    questions = d;
    isModalOpen = true;
    isShowQuestionModal = true;

    return null;
  }
</script>

<Theme persist bind:theme>
  <Header />
  <Content style="background: none; padding: 1rem">
    <Grid>
      <Row>
        <Column>
          <FileUploader
            bind:files
            labelTitle="Upload file"
            buttonLabel="Add file"
            labelDescription="Only CSV or TSV file is accepted."
            accept={[".csv", ".tsv"]}
            status="complete"
          />
        </Column>
        <Column>
          {#if isShowQuestionModal}
            <Button
              disabled={isModalOpen}
              on:click={() => {
                questionCurrentIndex = 0;
                currentAnswer = "";
                isModalOpen = true;
              }}>Show question</Button
            >
          {/if}
        </Column>
      </Row>
      <Row>
        <Column>
          <Modal
            preventCloseOnClickOutside={true}
            size="lg"
            bind:open={isModalOpen}
            modalHeading={modalHeader}
            primaryButtonText="Confirm"
            secondaryButtonText="Cancel"
            shouldSubmitOnEnter={false}
            on:click:button--secondary={() => (isModalOpen = false)}
            on:submit={() => openAnswerModal()}
          >
            <Grid>
              <Row>
                <p>{currentQuestion?.question}</p>
              </Row>
              <Row>
                <TextArea
                  bind:value={currentAnswer}
                  placeholder="Enter an answer..."
                />
              </Row>
            </Grid>
          </Modal>
          <Modal
            size="lg"
            bind:open={isAnswerModalOpen}
            modalHeading={`Answer: ${
              isCorrect(answerDiff) ? "✅ Correct" : "❌ Incorrect"
            }`}
            secondaryButtonText="Close"
            primaryButtonDisabled={true}
            on:click:button--secondary={() => (isAnswerModalOpen = false)}
            on:close={() => closeAnswerModal()}
          >
            <Grid>
              <Row>
                <div>
                  <p>
                    {#each answerDiff as part}
                      {#if part.added}
                        <span style="color: green">{part.value}</span>
                      {:else if part.removed}
                        <span style="color: red"><s>{part.value}</s></span>
                      {:else}
                        <span style="color: grey">{part.value}</span>
                      {/if}
                    {/each}
                  </p>
                </div>
              </Row>
            </Grid>
          </Modal>
        </Column>
      </Row>
    </Grid>
  </Content>
</Theme>
